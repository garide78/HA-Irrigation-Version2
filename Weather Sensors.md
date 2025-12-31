__Weather Sensors__

All weather sensors can be defined by the user.

There are six user defined sensors used in this system

1. Current Temperature
2. Forecast High Temperature Today
3. Total Rainfall so far today
4. Actual Rainfall yesterday

All the above sensors have a default based on [OpenWeatherMap](https://www.home-assistant.io/integrations/openweathermap/) but can be changed in the settings page.

5. Raining Now (optional)
If you want to prevent irrigation when it is rainig 'now' you need to proviude a `binary_senor`.

6. Weather Outlook (optional)
In order to have the weather outlook displayed you need to provide a sensor called `sensor.irrigation_weather_outlook`. Its state is some text and it can have either an icon or a picture defined.

Here is an example of the Weather Outlook sensor (this is mine and it will **NOT** work for you, it is *just an example*),
```
template:
  - sensor:
      #=== Irrigaton Weather Outlook
      #=== Creates a two line weather outlook with approprite icon 
      - name: Irrigation Weather Outlook
        unique_id: irrigation_weather_outlook
        state: >
          {% set current_conditions = states('sensor.weather_current_weather') | title %}
          {% set current_conditions = current_conditions.replace('Partlycloudy', 'Partly Cloudy') %}

          {% set max_high_temp = states('sensor.weather_forecast_high_temperature') %}
          {% set max_high_temp = max_high_temp if max_high_temp not in ['unknown', 'unavailable', 'none', None, ''] else 'N/A' %}
          {% set max_high_temp = max_high_temp | round(0) if max_high_temp is number else max_high_temp %}

          {% set will_rain_today = states('sensor.weather_will_it_rain_today') %}
          {% set will_rain_tomorrow = states('sensor.weather_will_it_rain_tomorrow') %}

          {% set total_rain_today = states('sensor.weather_forecast_total_rain_today') %}
          {% set total_rain_today = total_rain_today if total_rain_today not in ['unknown', 'unavailable', 'none', None, ''] else '0' %}

          {% set total_rain_tomorrow = states('sensor.weather_forecast_total_rain_tomorrow') %}
          {% set total_rain_tomorrow = total_rain_tomorrow if total_rain_tomorrow not in ['unknown', 'unavailable', 'none', None, ''] else '0' %}

          {% set probability_of_rain_today = states('sensor.weather_probability_of_rain_today') %}
          {% set probability_of_rain_today = probability_of_rain_today if probability_of_rain_today not in ['unknown', 'unavailable', 'none', None, ''] else '0' %}

          {% set probability_of_rain_tomorrow = states('sensor.weather_probability_of_rain_tomorrow') %}
          {% set probability_of_rain_tomorrow = probability_of_rain_tomorrow if probability_of_rain_tomorrow not in ['unknown', 'unavailable', 'none', None, ''] else '0' %}

          {% set rain_today = total_rain_today ~ 'mm / ' ~ probability_of_rain_today ~ '%' %}
          {% set rain_tomorrow = total_rain_tomorrow ~ 'mm / ' ~ probability_of_rain_tomorrow ~ '%' %}

          {% set outlook = 'Outlook: ' ~ current_conditions ~ ' High Temp ' ~ max_high_temp ~ '°C<br>' %}

          {% set rain_today_bool = will_rain_today in ['1', 'yes', 'true'] %}
          {% set rain_tomorrow_bool = will_rain_tomorrow in ['1', 'yes', 'true'] %}

          {% if rain_today_bool and rain_tomorrow_bool %}
            {% set outlook = outlook ~ 'Rain Today (' ~ rain_today ~ ') & Tomorrow (' ~ rain_tomorrow ~ ')' %}
          {% elif rain_today_bool %}
            {% set outlook = outlook ~ 'Rain Today (' ~ rain_today ~ '), none tomorrow' %}
          {% elif rain_tomorrow_bool %}
            {% set outlook = outlook ~ 'Rain Tomorrow (' ~ rain_tomorrow ~ ')' %}
          {% else %}
            {% set outlook = outlook ~ 'No rain forecast today or tomorrow.' %}
          {% endif %}

          {{ outlook }}
        picture: >
          {% set current = state_attr('sensor.weather_api_current', 'current') %}
          {% set icon = current.condition.icon.split('/')[-1] if current is defined and current.condition is defined and current.condition.icon is defined else 'unknown.png' %}
          {% set is_day = true if state_attr('sun.sun', 'elevation') | float > 0 else false %}
          {% if is_day %}
            {{ '/local/icons/weather_icons/weather_api_day/' ~ icon }}
          {% else %}
            {{ '/local/icons/weather_icons/weather_api_night/' ~ icon }}
          {% endif %}
        availability: >
          {% set weather_sensors = expand(
            "sensor.weather_forecast_high_temperature",
            "sensor.weather_will_it_rain_today",
            "sensor.weather_will_it_rain_tomorrow",
            "sensor.weather_forecast_total_rain_today",
            "sensor.weather_forecast_total_rain_tomorrow",
            "sensor.weather_probability_of_rain_today",
            "sensor.weather_probability_of_rain_tomorrow",
            "sensor.weather_current_weather"
          ) %}
          {{ weather_sensors | selectattr('state','in', ['unknown','unavailable']) | list | length == 0 }}
        attributes:
          current_conditions: >
            {{ states('sensor.weather_current_weather') | title }}
          max_high_temp: >
            {{ states('sensor.weather_forecast_high_temperature') | round() }}
          will_rain_today: >
            {{ states('sensor.weather_will_it_rain_today') }}
          will_rain_tomorrow: >
            {{ states('sensor.weather_will_it_rain_tomorrow') }}
          total_rain_today: >
            {{ states('sensor.weather_forecast_total_rain_today') }}
          total_rain_tomorrow: >
            {{ states('sensor.weather_forecast_total_rain_tomorrow') }}
          probability_of_rain_today: >
            {{ states('sensor.weather_probability_of_rain_today') }}
          probability_of_rain_tomorrow: >
            {{ states('sensor.weather_probability_of_rain_tomorrow') }}
          rain_today: >
            {{ states('sensor.weather_forecast_total_rain_today') ~ 'mm / ' ~ states('sensor.weather_probability_of_rain_today') ~ '%' }}
          rain_tomorrow: >
            {{ states('sensor.weather_forecast_total_rain_tomorrow') ~ 'mm / ' ~ states('sensor.weather_probability_of_rain_tomorrow') ~ '%' }}

```

__Night or Day__

The weather graphs are shaded using this sensor

```
template:
  - sensor:
      - name: Night or Day
        unique_id: night_or_day
        state: >
          {% if is_state('sun.sun', 'above_horizon') %}
            0
          {% else %}
            1
          {% endif %}
        icon: mdi:power-sleep
```
