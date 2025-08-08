---
description: Documentation for Weather Forecast
title: Weather Forecast
---

# Weather Forecast

This example demonstrates how to create a Flutter application that utilizes the Maps SDK for Flutter to display a weather forecast on a map. The application initializes the Maps SDK for Flutter and provides a user interface to show current, hourly, and daily weather forecasts.

## How It Works

- Main App Setup : The main app initializes GemKit and displays a map.

- Weather Forecast Page : Users can navigate through current, hourly and daily forecasts for a specific location.

### UI and Map Integration

The main application consists of a simple user interface that displays a map along with a weather forecast option. The user can tap on the weather icon in the app bar to navigate to a detailed weather forecast page.
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Weather Forecast',
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
        title: const Text('Weather Forecast', style: TextStyle(color: Colors.white)),
        actions: [
          IconButton(
            onPressed: () => _onWeatherForecastTap(context),
            icon: Icon(
              Icons.sunny,
              color: Colors.white,
            ),
          ),
        ],
      ),
      body: GemMap(appAuthorization: projectApiToken),
    );
  }

  void _onWeatherForecastTap(BuildContext context) {
    Navigator.of(context).push(MaterialPageRoute<dynamic>(
      builder: (context) => WeatherForecastPage(),
    ));
  }
}
```

### Weather Forecast Page

The WeatherForecastPage displays the weather forecast options, allowing users to switch between current, hourly, and daily forecasts. This code implements the WeatherForecastPage , which manages the state of the currently selected weather tab and displays the appropriate forecast information.
```dart
class WeatherForecastPage extends StatefulWidget {
  const WeatherForecastPage({super.key});

  @override
  State<WeatherForecastPage> createState() => _WeatherForecastPageState();
}

class _WeatherForecastPageState extends State<WeatherForecastPage> {
  // Variable to track the selected weather tab, defaulting to 'now'
  WeatherTab _weatherTab = WeatherTab.now;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        automaticallyImplyLeading: true,
        foregroundColor: Colors.white,
        title: const Text(
          "Weather Forecast",
          style: TextStyle(color: Colors.white),
        ),
        backgroundColor: Colors.deepPurple[900],
      ),
      body: Padding(
        padding: const EdgeInsets.all(15.0),
        child: Column(
          children: [
            // Tab buttons for 'Now', 'Hourly', and 'Daily' forecasts
            SizedBox(
              height: 40.0,
              child: Row(
                children: [
                  Expanded(
                    child: InkWell(
                      child: Center(child: Text("Now")),
                      onTap:
                          () => setState(() {
                            _weatherTab = WeatherTab.now;
                          }),
                    ),
                  ),
                  Expanded(
                    child: InkWell(
                      child: Center(child: Text("Hourly")),
                      onTap:
                          () => setState(() {
                            _weatherTab = WeatherTab.hourly;
                          }),
                    ),
                  ),
                  Expanded(
                    child: InkWell(
                      child: Center(child: Text("Daily")),
                      onTap:
                          () => setState(() {
                            _weatherTab = WeatherTab.daily;
                          }),
                    ),
                  ),
                ],
              ),
            ),

            // Display the selected forecast page
            Expanded(
              child: Builder(
                builder: (context) {
                  if (_weatherTab == WeatherTab.now) {
                    return FutureBuilder(
                      future: _getCurrentForecast(),
                      builder: (context, snapshot) {
                        if (snapshot.connectionState ==
                            ConnectionState.waiting) {
                          return Center(child: CircularProgressIndicator());
                        }
                        if (!snapshot.hasData) {
                          return Center(
                            child: Text("Error loading current forecast."),
                          );
                        }

                        return ForecastNowPage(
                          condition: snapshot.data!,
                          landmarkName: "Paris",
                        );
                      },
                    );
                  } else if (_weatherTab == WeatherTab.hourly) {
                    return FutureBuilder(
                      future: _getHourlyForecast(),
                      builder: (context, snapshot) {
                        if (snapshot.connectionState ==
                            ConnectionState.waiting) {
                          return Center(child: CircularProgressIndicator());
                        }
                        if (!snapshot.hasData) {
                          return Center(
                            child: Text("Error loading hourly forecast."),
                          );
                        }

                        return ForecastHourlyPage(
                          locationForecasts: snapshot.data!,
                        );
                      },
                    );
                  } else if (_weatherTab == WeatherTab.daily) {
                    return FutureBuilder(
                      future: _getDailyForecast(),
                      builder: (context, snapshot) {
                        if (snapshot.connectionState ==
                            ConnectionState.waiting) {
                          return Center(child: CircularProgressIndicator());
                        }

                        if (!snapshot.hasData) {
                          return Center(
                            child: Text("Error loading daily forecast."),
                          );
                        }

                        return ForecastDailyPage(
                          locationForecasts: snapshot.data!,
                        );
                      },
                    );
                  }
                  return Container();
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
```

### Getting Current, Hourly and Daily Forecasts

The following methods retrieve hourly and daily forecasts. These methods use the WeatherService to fetch current, hourly and daily weather data for a specified location.
```dart
 Future<LocationForecast> _getCurrentForecast() async {
    final locationCoordinates = Coordinates(
      latitude: 48.864716,
      longitude: 2.349014,
    );
    final weatherCurrentCompleter = Completer<List<LocationForecast>?>();

    WeatherService.getCurrent(
      coords: [locationCoordinates],
      onComplete: (err, result) async {
        weatherCurrentCompleter.complete(result);
      },
    );

    final currentForecast = await weatherCurrentCompleter.future;

    return currentForecast!.first;
  }

  Future<List<LocationForecast>> _getHourlyForecast() async {
    final locationCoordinates = Coordinates(
      latitude: 48.864716,
      longitude: 2.349014,
    );
    final weatherHourlyCompleter = Completer<List<LocationForecast>?>();

    WeatherService.getHourlyForecast(
      hours: 24,
      coords: [locationCoordinates],
      onComplete: (err, result) async {
        weatherHourlyCompleter.complete(result);
      },
    );

    final currentForecast = await weatherHourlyCompleter.future;

    return currentForecast!;
  }

  Future<List<LocationForecast>> _getDailyForecast() async {
    final locationCoordinates = Coordinates(
      latitude: 48.864716,
      longitude: 2.349014,
    );
    final weatherDailyCompleter = Completer<List<LocationForecast>?>();

    WeatherService.getDailyForecast(
      days: 10,
      coords: [locationCoordinates],
      onComplete: (err, result) async {
        weatherDailyCompleter.complete(result);
      },
    );

    final currentForecast = await weatherDailyCompleter.future;

    return currentForecast!;
  }
```

### Hourly Forecast Page and Item

The ForecastHourlyPage displays the hourly weather forecasts. Below is the implementation of this page. This code defines the ForecastHourlyPage , which presents a list of hourly weather forecasts for the user.
```dart
class ForecastHourlyPage extends StatelessWidget {
  final List<LocationForecast> locationForecasts;

  const ForecastHourlyPage({super.key, required this.locationForecasts});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: MediaQuery.of(context).size.height * 0.8,
      child: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: locationForecasts.first.forecast.length,
              itemBuilder: (context, index) {
                return WeatherForecastHourlyItem(
                  condition: locationForecasts.first.forecast[index],
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

class WeatherForecastHourlyItem extends StatelessWidget {
  /// The weather conditions for a specific hour.
  final Conditions condition;

  const WeatherForecastHourlyItem({super.key, required this.condition});

  @override
  Widget build(BuildContext context) {
    // Extracting the image and temperature information from the condition.
    final conditionImage = condition.img;
    final tempHigh =
        condition.params
            .where((element) => element.type == "Temperature")
            .first;

    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(condition.getFormattedHour()),
              Text(condition.getFormattedDate()),
            ],
          ),
          conditionImage.isValid ? Image.memory(conditionImage.getRenderableImageBytes()!) : SizedBox(),
          Text("${tempHigh.value} ${tempHigh.unit}"),
        ],
      ),
    );
  }
}
```

## Daily Forecast Page and Item

The ForecastDailyPage displays the daily weather forecasts. Below is the implementation of this page. This code defines the ForecastDailyPage , which presents a list of daily weather forecasts for the user.
```dart
class ForecastDailyPage extends StatelessWidget {
  final List<LocationForecast> locationForecasts;

  const ForecastDailyPage({super.key, required this.locationForecasts});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: MediaQuery.of(context).size.height * 0.8,
      child: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: locationForecasts.first.forecast.length,
              itemBuilder: (context, index) {
                return WeatherForecastDailyItem(
                  condition: locationForecasts.first.forecast[index],
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

class WeatherForecastDailyItem extends StatelessWidget {
  /// The weather conditions for a specific day.
  final Conditions condition;

  const WeatherForecastDailyItem({super.key, required this.condition});

  @override
  Widget build(BuildContext context) {
    final conditionImage = condition.img;
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(_getWeekdayString(condition.stamp.weekday)),
              Text(condition.getFormattedDate()),
            ],
          ),
          conditionImage.isValid ? Image.memory(conditionImage.getRenderableImageBytes()!) : SizedBox(),
          Row(children: [Text(condition.getFormattedTemperature())]),
        ],
      ),
    );
  }

  // Converts a weekday index into a string representation.
  String _getWeekdayString(int weekday) {
    const weekdays = [
      'Monday',
      'Tuesday',
      'Wednesday',
      'Thursday',
      'Friday',
      'Saturday',
      'Sunday',
    ];
    return weekdays[weekday - 1];
  }
}
```

## Forecast Now Page
```dart
class ForecastNowPage extends StatelessWidget {
  /// Holds the weather condition and forecast data.
  final LocationForecast condition;

  /// The name of the landmark or location for which the weather forecast is displayed.
  final String landmarkName;

  const ForecastNowPage({
    super.key,
    required this.condition,
    required this.landmarkName,
  });

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            SizedBox(
              child: condition.forecast.first.img.isValid ? Image.memory(condition.forecast.first.img.getRenderableImageBytes()!) : SizedBox(),
            
            ),
            Text(condition.forecast.first.description),
          ],
        ),
        Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            ListView(
              shrinkWrap: true,
              children: [
                Column(
                  children: [
                    Text(landmarkName, style: TextStyle(fontSize: 20.0)),
                    Text("Updated at ${condition.getFormattedHour()}"),
                  ],
                ),
                for (final param in condition.forecast.first.params)
                  Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Text(param.name),
                        Row(
                          children: [
                            Text(
                              param.type == "Sunrise" || param.type == "Sunset"
                                  ? param.getFormattedHour()
                                  : param.value.toString(),
                            ),
                            Text(param.unit),
                          ],
                        ),
                      ],
                    ),
                  ),
              ],
            ),
          ],
        ),
      ],
    );
  }
}
```


