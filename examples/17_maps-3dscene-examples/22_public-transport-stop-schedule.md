---
description: Documentation for Public Transport Stop Schedule
title: Public Transport Stop Schedule
---

# Public Transit Stop Schedule

This example showcases how to build a Flutter app featuring an interactive map. Users can select public transport points of interest (POIs) to access detailed information, including departure times, route names, and the status of transport vehicles, such as whether they have departed.

## How it works

The example app includes the following features:

- Display a map.

- Select public transport stations from the map.

- Display public transport trip information.

- Show departure times for each public transport trip.

### UI and Map Integration

The following code creates a user interface with an interactive `GemMap` and an app bar.
```dart
const projectApiToken = String.fromEnvironment('GEM_TOKEN');

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Public Transit Stops',
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late GemMapController _mapController;
  PTStopInfo? _selectedPTStop;
  Coordinates? _selectedPTStopCoords;
  @override
  void dispose() {
    GemKit.release();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.deepPurple[900],
        title: const Text('Public Transit Stops', style: TextStyle(color: Colors.white)),
      ),
      body: Stack(
        children: [
          GemMap(
            key: ValueKey("GemMap"),
            appAuthorization: projectApiToken,
            onMapCreated: (controller) => _onMapCreated(controller),
          ),
          if (_selectedPTStop != null)
            Positioned(
              bottom: 0,
              left: 0,
              right: 0,
              child: SizedBox(
                height: MediaQuery.of(context).size.height * 0.6,
                child: FutureBuilder(
                    future: getLocalTime(_selectedPTStopCoords!),
                    builder: (context, snapshot) {
                      if (snapshot.connectionState == ConnectionState.waiting) return Container();
                      return PublicTransitStopPanel(
                        ptStopInfo: _selectedPTStop!,
                        localTime: snapshot.data!,
                        onCloseTap: () => setState(
                          () {
                            _selectedPTStop = null;
                            _selectedPTStopCoords = null;
                          },
                        ),
                      );
                    }),
              ),
            )
        ],
      ),
    );
  }

  void _onMapCreated(GemMapController controller) async {
    // Save controller for further usage.
    _mapController = controller;

    _mapController.registerLongPressCallback((pos) async {
      // Update the cursor screen position
      await _mapController.setCursorScreenPosition(pos);

      // Get the public transit overlay items at that position
      final items = _mapController.cursorSelectionOverlayItemsByType(CommonOverlayId.publicTransport);
      final coords = _mapController.transformScreenToWgs(pos);

      for (final OverlayItem item in items) {
        // Get the stop information
        final ptStopInfo = await item.getPTStopInfo();
        if (ptStopInfo != null) {
          setState(() {
            _selectedPTStop = ptStopInfo;
            _selectedPTStopCoords = coords;
          });
        }
      }
    });
  }
}
```

### Public Transport Stop Panel
```dart
class PublicTransitStopPanel extends StatefulWidget {
  final PTStopInfo ptStopInfo;
  final DateTime localTime;
  final VoidCallback onCloseTap;

  const PublicTransitStopPanel({
    super.key,
    required this.ptStopInfo,
    required this.localTime,
    required this.onCloseTap,
  });

  @override
  State<PublicTransitStopPanel> createState() => _PublicTransitStopPanelState();
}

class _PublicTransitStopPanelState extends State<PublicTransitStopPanel> {
  PTTrip? _selectedTrip;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Container(
          width: double.infinity,
          color: Colors.white,
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              if (_selectedTrip != null)
                IconButton(
                  icon: const Icon(Icons.arrow_back),
                  onPressed: () => setState(() => _selectedTrip = null),
                )
              else
                const SizedBox(width: 48),
              Text(
                _selectedTrip == null ? 'Select a Trip' : 'Stops for ${_selectedTrip!.route.routeShortName}',
              ),
              IconButton(
                icon: const Icon(Icons.close),
                onPressed: widget.onCloseTap,
              ),
            ],
          ),
        ),

        // Body: either list of trips or list of stops
        Expanded(
          child: Container(
            color: Colors.white,
            child: _selectedTrip == null
                ? ListView.separated(
                    itemCount: widget.ptStopInfo.trips.length,
                    itemBuilder: (context, index) {
                      final trip = widget.ptStopInfo.trips[index];
                      return Padding(
                        padding: const EdgeInsets.all(8.0),
                        child: PTLineListItem(
                          localCurrentTime: widget.localTime,
                          ptTrip: trip,
                          onTap: () {
                            setState(() {
                              _selectedTrip = trip;
                            });
                          },
                        ),
                      );
                    },
                    separatorBuilder: (_, __) => const Divider(
                      height: 1,
                      thickness: 1,
                      indent: 16,
                      endIndent: 16,
                    ),
                  )
                : StopsPanel(
                    stopTimes: _selectedTrip!.stopTimes,
                    localTime: widget.localTime,
                    onCloseTap: widget.onCloseTap,
                  ),
          ),
        ),
      ],
    );
  }
}

class PTLineListItem extends StatelessWidget {
  final PTTrip ptTrip;
  final DateTime localCurrentTime;
  final VoidCallback onTap;

  const PTLineListItem({super.key, required this.ptTrip, required this.localCurrentTime, required this.onTap});
  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap,
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Row(
            children: [
              Icon(getTransportIcon(ptTrip.route.routeType)),
              SizedBox(width: 7.0),
              Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Container(
                    padding: EdgeInsets.symmetric(vertical: 4.0, horizontal: 15.0),
                    decoration: BoxDecoration(
                      color: ptTrip.route.routeColor,
                      borderRadius: BorderRadius.circular(14.0), // adjust radius as you like
                    ),
                    child: Text(
                      ptTrip.route.routeShortName ?? "None",
                      style: TextStyle(color: ptTrip.route.routeTextColor, fontWeight: FontWeight.w500),
                    ),
                  ),
                  Text(
                    ptTrip.route.heading ?? ptTrip.route.routeLongName ?? "Nan",
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  Text(
                    ptTrip.departureTime != null
                        ? 'Scheduled • ${DateFormat('H:mm').format(ptTrip.departureTime!)}'
                        : 'Scheduled • ',
                  ),
                ],
              ),
            ],
          ),
          Text(calculateTimeDifference(localCurrentTime, ptTrip))
        ],
      ),
    );
  }
}
```

### Departure Times Panel
```dart
class DepartureTimesPanel extends StatelessWidget {
  final List<PTStopTime> stopTimes;
  final DateTime localTime;
  final VoidCallback onCloseTap;
  const DepartureTimesPanel({super.key, required this.stopTimes, required this.localTime, required this.onCloseTap});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // the scrollable list with dividers
        Expanded(
          child: Container(
            color: Colors.white,
            child: ListView.separated(
              itemCount: stopTimes.length,
              itemBuilder: (context, index) => Padding(
                padding: const EdgeInsets.all(8.0),
                child: DepartureTimesListItem(
                  stop: stopTimes[index],
                  localCurrentTime: localTime,
                ),
              ),
              separatorBuilder: (context, index) => const Divider(
                height: 1,
                thickness: 1,
                indent: 16, // optional: inset the divider from the left
                endIndent: 16, // optional: inset the divider from the right
              ),
            ),
          ),
        ),
      ],
    );
  }
}

class DepartureTimesListItem extends StatelessWidget {
  final PTStopTime stop;
  final DateTime localCurrentTime;
  const DepartureTimesListItem({super.key, required this.stop, required this.localCurrentTime});

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceBetween,
      children: [
        Expanded(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                stop.stopName,
                maxLines: 1,
                overflow: TextOverflow.ellipsis,
              ),
              if (stop.departureTime != null)
                Text(stop.departureTime!.isAfter(localCurrentTime) ? "Scheduled" : "Departed"),
            ],
          ),
        ),
        SizedBox(width: 8),
        Text(
          stop.departureTime != null ? DateFormat('H:mm').format(stop.departureTime!) : '-',
        ),
      ],
    );
  }
}
```

### Utility Functions
```dart
// Retrieves the local time for the region corresponding to the given coordinates.
Future<DateTime> getLocalTime(Coordinates referenceCoords) async {
  final completer = Completer<TimezoneResult>();

  TimezoneService.getTimezoneInfoFromCoordinates(
    coords: referenceCoords,
    time: DateTime.now(),
    onComplete: (error, result) {
      if (error == GemError.success) completer.complete(result);
    },
  );

  final timezoneResult = await completer.future;

  return timezoneResult.localTime;
}

IconData getTransportIcon(PTRouteType type) {
  switch (type) {
    case PTRouteType.bus:
      return Icons.directions_bus;
    case PTRouteType.underground:
      return Icons.directions_subway;
    case PTRouteType.railway:
      return Icons.directions_railway;
    case PTRouteType.tram:
      return Icons.directions_bus_filled;
    case PTRouteType.waterTransport:
      return Icons.directions_boat;
    case PTRouteType.misc:
      return Icons.miscellaneous_services;
  }
}

// Computes how many minutes remain until the PTTrip’s scheduled departure.
String calculateTimeDifference(DateTime localCurrentTime, PTTrip ptTrip) {
  return ptTrip.departureTime != null ? '${ptTrip.departureTime!.difference(localCurrentTime).inMinutes} min' : '–';
}
```


