title: Weather Station Dashboard
views:
  - title: Weather Overview
    path: weather_overview
    icon: mdi:weather-cloudy
    type: sections
    sections:
      - type: grid
        columns: 3
        cards:
          - type: custom:mushroom-template-card
            primary: '{{ states(''sensor.gw2000b_outdoor_temperature'') }}°C'
            secondary: Feels like {{ states('sensor.gw2000b_feels_like_temperature') }}°C
            icon: mdi:thermometer
            layout: vertical
            fill_container: true
            tap_action:
              action: more-info
              entity: sensor.gw2000b_outdoor_temperature
            icon_color: >-
              {% set temp = states('sensor.gw2000b_outdoor_temperature') | float
              %} {% if temp > 25 %}red {% elif temp > 20 %}orange {% elif temp >
              15 %}yellow {% elif temp > 10 %}green {% else %}blue{% endif %}
          - type: custom:mushroom-template-card
            primary: '{{ states(''sensor.gw2000b_humidity'') }}%'
            secondary: Dewpoint {{ states('sensor.gw2000b_dewpoint') }}°C
            icon: mdi:water-percent
            layout: vertical
            fill_container: true
            tap_action:
              action: more-info
              entity: sensor.gw2000b_humidity
            icon_color: >-
              {% set humidity = states('sensor.gw2000b_humidity') | float %} {%
              if humidity > 85 %}red {% elif humidity > 70 %}orange {% elif
              humidity > 60 %}yellow {% else %}green{% endif %}
          - type: custom:mushroom-template-card
            primary: '{{ states(''sensor.gw2000b_rain_rate_piezo'') }} mm/hr'
            secondary: |
              {% if states('sensor.gw2000b_rain_rate_piezo') | float > 0 %}
                Rain Detected!
              {% else %}
                No Rain
              {% endif %}
            icon: mdi:weather-rainy
            layout: vertical
            fill_container: true
            icon_color: |-
              {% if states('sensor.gw2000b_rain_rate_piezo') | float > 0 %}
                red
              {% else %}
                green
              {% endif %}
            entity: sensor.gw2000b_rain_rate_piezo
            tap_action:
              action: more-info
          - type: custom:mushroom-template-card
            primary: Pressure Trends
            secondary: >
              {{ states('sensor.air_pressure_trend') }} | {{
              states('sensor.pressure_trend') }}
            icon: mdi:trending-up
            layout: vertical
            fill_container: true
            icon_color: ''
            grid_options:
              columns: 6
              rows: 2
          - type: picture-entity
            entity: camera.backyard
            name: Backyard Live
            show_state: true
            show_name: true
            camera_view: auto
      - type: grid
        columns: 2
        cards:
          - type: custom:mini-graph-card
            entities:
              - entity: sensor.gw2000b_outdoor_temperature
                name: Temperature
                color: '#e74c3c'
              - entity: sensor.gw2000b_feels_like_temperature
                name: Feels Like
                color: '#3498db'
            hours_to_show: 24
            points_per_hour: 4
            line_width: 2
            hour24: true
            show:
              labels: true
              name: true
              legend: true
          - type: custom:apexcharts-card
            graph_span: 24h
            header:
              show: true
              title: Pressure Trends
            series:
              - entity: sensor.gw2000b_absolute_pressure
                name: Absolute
                stroke_width: 2
                curve: smooth
              - entity: sensor.gw2000b_relative_pressure
                name: Relative
                stroke_width: 2
                curve: smooth
            yaxis:
              - min: 950
                max: 1050
                apex_config:
                  tickAmount: 5
      - type: grid
        columns: 2
        cards:
          - type: iframe
            url: >-
              https://api.weather.com/v2/maps/dynamic?geocode=51.70,-2.50&h=320&w=568&lod=9&product=twcRadarHcMosaic&map=light&format=jpg&language=en-GB&apiKey=71f92ea9dd2f4790b92ea9dd2f779061&a=0
            aspect_ratio: 50%
            title: Current Radar
          - type: vertical-stack
            cards:
              - type: vertical-stack
                cards:
                  - type: custom:mushroom-template-card
                    primary: >-
                      {{ state_attr('binary_sensor.meteoalarm', 'headline') or
                      'No Active Alerts' }}
                    secondary: >-
                      {% set desc = state_attr('binary_sensor.meteoalarm',
                      'description') %} {% if desc %}
                        {{ desc }}
                      {% else %}
                        Currently there are no weather alerts for your area
                      {% endif %}
                    icon: |-
                      {% if is_state('binary_sensor.meteoalarm', 'on') %}
                        {% set awareness = state_attr('binary_sensor.meteoalarm', 'awareness_type') | lower %}
                        {% if awareness == 'wind' %}mdi:weather-windy
                        {% elif awareness == 'snow_ice' %}mdi:weather-snowy-heavy
                        {% elif awareness == 'thunderstorms' %}mdi:weather-lightning
                        {% elif awareness == 'fog' %}mdi:weather-fog
                        {% elif awareness == 'high_temperature' %}mdi:thermometer-high
                        {% elif awareness == 'low_temperature' %}mdi:thermometer-low
                        {% elif awareness == 'coastal_event' %}mdi:waves
                        {% elif awareness == 'forest_fire' %}mdi:fire-alert
                        {% elif awareness == 'avalanches' %}mdi:avalanche
                        {% elif awareness == 'rain' %}mdi:weather-pouring
                        {% elif awareness == 'flooding' %}mdi:home-flood
                        {% else %}mdi:alert-circle
                        {% endif %}
                      {% else %}
                        mdi:shield-check
                      {% endif %}
                    layout: vertical
                    fill_container: true
                    icon_color: |-
                      {% if is_state('binary_sensor.meteoalarm', 'on') %}
                        {% set level = state_attr('binary_sensor.meteoalarm', 'awareness_level') | int %}
                        {% if level == 2 %}yellow
                        {% elif level == 3 %}orange
                        {% elif level == 4 %}red
                        {% else %}green
                        {% endif %}
                      {% else %}
                        green
                      {% endif %}
                    tap_action:
                      action: more-info
                    hold_action:
                      action: url
                      url_path: https://www.meteoalarm.eu
          - type: tile
            entity: sensor.lydney_weather_alert_weathercom
            grid_options:
              columns: full
          - type: tile
            entity: sensor.lydney_weather_insight_weathercom
            grid_options:
              columns: full
          - type: tile
            entity: weather.forecast_home
            grid_options:
              columns: full
      - type: grid
        columns: 2
        cards:
          - type: custom:mini-graph-card
            entities:
              - entity: sensor.gw2000b_wind_speed
                name: Wind Speed
                color: '#3498db'
              - entity: sensor.gw2000b_wind_gust
                name: Wind Gust
                color: '#e74c3c'
            hours_to_show: 12
            points_per_hour: 6
            line_width: 2
            hour24: true
            show:
              labels: true
              points: true
              legend: true
          - type: custom:mushroom-template-card
            primary: Wind Direction
            secondary: >-
              {% set direction = states('sensor.gw2000b_wind_direction') | float
              %} {% set compass = [
                {'min': 348.75, 'max': 360, 'dir': 'N'},
                {'min': 0, 'max': 11.25, 'dir': 'N'},
                {'min': 11.25, 'max': 33.75, 'dir': 'NNE'},
                {'min': 33.75, 'max': 56.25, 'dir': 'NE'},
                {'min': 56.25, 'max': 78.75, 'dir': 'ENE'},
                {'min': 78.75, 'max': 101.25, 'dir': 'E'},
                {'min': 101.25, 'max': 123.75, 'dir': 'ESE'},
                {'min': 123.75, 'max': 146.25, 'dir': 'SE'},
                {'min': 146.25, 'max': 168.75, 'dir': 'SSE'},
                {'min': 168.75, 'max': 191.25, 'dir': 'S'},
                {'min': 191.25, 'max': 213.75, 'dir': 'SSW'},
                {'min': 213.75, 'max': 236.25, 'dir': 'SW'},
                {'min': 236.25, 'max': 258.75, 'dir': 'WSW'},
                {'min': 258.75, 'max': 281.25, 'dir': 'W'},
                {'min': 281.25, 'max': 303.75, 'dir': 'WNW'},
                {'min': 303.75, 'max': 326.25, 'dir': 'NW'},
                {'min': 326.25, 'max': 348.75, 'dir': 'NNW'}
              ] %} {% set curr_dir = 'N' %} {% for item in compass %}
                {% if direction >= item.min and direction < item.max %}
                  {% set curr_dir = item.dir %}
                {% endif %}
              {% endfor %} {{ direction }}° {{ curr_dir }}
            icon: >
              {% set direction = states('sensor.gw2000b_wind_direction') | float
              %}

              {% set icon_map = {
                'N': 'arrow-up',
                'NNE': 'arrow-up',
                'NE': 'arrow-top-right',
                'ENE': 'arrow-top-right',
                'E': 'arrow-right',
                'ESE': 'arrow-right',
                'SE': 'arrow-bottom-right',
                'SSE': 'arrow-bottom-right',
                'S': 'arrow-down',
                'SSW': 'arrow-down',
                'SW': 'arrow-bottom-left',
                'WSW': 'arrow-bottom-left',
                'W': 'arrow-left',
                'WNW': 'arrow-left',
                'NW': 'arrow-top-left',
                'NNW': 'arrow-top-left'
              } %}

              {% set compass = [
                {'min': 348.75, 'max': 360, 'dir': 'N'},
                {'min': 0, 'max': 11.25, 'dir': 'N'},
                {'min': 11.25, 'max': 33.75, 'dir': 'NNE'},
                {'min': 33.75, 'max': 56.25, 'dir': 'NE'},
                {'min': 56.25, 'max': 78.75, 'dir': 'ENE'},
                {'min': 78.75, 'max': 101.25, 'dir': 'E'},
                {'min': 101.25, 'max': 123.75, 'dir': 'ESE'},
                {'min': 123.75, 'max': 146.25, 'dir': 'SE'},
                {'min': 146.25, 'max': 168.75, 'dir': 'SSE'},
                {'min': 168.75, 'max': 191.25, 'dir': 'S'},
                {'min': 191.25, 'max': 213.75, 'dir': 'SSW'},
                {'min': 213.75, 'max': 236.25, 'dir': 'SW'},
                {'min': 236.25, 'max': 258.75, 'dir': 'WSW'},
                {'min': 258.75, 'max': 281.25, 'dir': 'W'},
                {'min': 281.25, 'max': 303.75, 'dir': 'WNW'},
                {'min': 303.75, 'max': 326.25, 'dir': 'NW'},
                {'min': 326.25, 'max': 348.75, 'dir': 'NNW'}
              ] %}

              {% set curr_dir = 'N' %}

              {% for item in compass %}
                {% if direction >= item.min and direction < item.max %}
                  {% set curr_dir = item.dir %}
                {% endif %}
              {% endfor %}

              mdi:{{ icon_map[curr_dir] }}
            layout: vertical
            fill_container: true
            icon_color: >
              {% set direction = states('sensor.gw2000b_wind_direction') | float
              %}

              {% set colors = {
                'N': 'blue',
                'NNE': 'blue',
                'NE': 'purple',
                'ENE': 'purple',
                'E': 'red',
                'ESE': 'red',
                'SE': 'orange',
                'SSE': 'orange',
                'S': 'yellow',
                'SSW': 'yellow',
                'SW': 'green',
                'WSW': 'green',
                'W': 'cyan',
                'WNW': 'cyan',
                'NW': 'blue',
                'NNW': 'blue'
              } %}

              {% set compass = [
                {'min': 348.75, 'max': 360, 'dir': 'N'},
                {'min': 0, 'max': 11.25, 'dir': 'N'},
                {'min': 11.25, 'max': 33.75, 'dir': 'NNE'},
                {'min': 33.75, 'max': 56.25, 'dir': 'NE'},
                {'min': 56.25, 'max': 78.75, 'dir': 'ENE'},
                {'min': 78.75, 'max': 101.25, 'dir': 'E'},
                {'min': 101.25, 'max': 123.75, 'dir': 'ESE'},
                {'min': 123.75, 'max': 146.25, 'dir': 'SE'},
                {'min': 146.25, 'max': 168.75, 'dir': 'SSE'},
                {'min': 168.75, 'max': 191.25, 'dir': 'S'},
                {'min': 191.25, 'max': 213.75, 'dir': 'SSW'},
                {'min': 213.75, 'max': 236.25, 'dir': 'SW'},
                {'min': 236.25, 'max': 258.75, 'dir': 'WSW'},
                {'min': 258.75, 'max': 281.25, 'dir': 'W'},
                {'min': 281.25, 'max': 303.75, 'dir': 'WNW'},
                {'min': 303.75, 'max': 326.25, 'dir': 'NW'},
                {'min': 326.25, 'max': 348.75, 'dir': 'NNW'}
              ] %}

              {% set curr_dir = 'N' %}

              {% for item in compass %}
                {% if direction >= item.min and direction < item.max %}
                  {% set curr_dir = item.dir %}
                {% endif %}
              {% endfor %}

              {{ colors[curr_dir] }}
            tap_action:
              action: more-info
            grid_options:
              columns: full
              rows: 2
          - type: gauge
            entity: sensor.gw2000b_wh90_battery
            max: 5
            min: 0
            needle: false
            grid_options:
              columns: 6
              rows: auto
          - type: gauge
            entity: sensor.gw2000b_wh90_capacitor
            max: 5
            min: 0
            grid_options:
              columns: 6
              rows: auto
      - type: grid
        columns: 2
        cards:
          - type: custom:apexcharts-card
            graph_span: 12h
            header:
              show: true
              title: Solar Radiation & UV
            series:
              - entity: sensor.gw2000b_solar_radiation
                name: Solar Radiation
                stroke_width: 2
                curve: smooth
              - entity: sensor.gw2000b_uv_index
                name: UV Index
                stroke_width: 2
                curve: smooth
                group_by:
                  func: avg
                  duration: 10min
              - entity: sensor.gw2000b_solar_lux
                name: Solar Lux
                stroke_width: 2
                curve: smooth
                group_by:
                  func: avg
                  duration: 10min
          - type: custom:weather-chart-card
            entity: weather.forecast_home
            show_main: true
            show_temperature: true
            show_current_condition: true
            show_attributes: true
            show_time: false
            show_time_seconds: false
            show_day: false
            show_date: false
            show_humidity: true
            show_pressure: true
            show_wind_direction: true
            show_wind_speed: true
            show_sun: true
            show_feels_like: false
            use_12hour_format: false
            icons_size: 25
            animated_icons: false
            icon_style: style1
            forecast:
              precipitation_type: rainfall
              show_probability: false
              labels_font_size: '11'
              precip_bar_size: '100'
              style: style1
              show_wind_forecast: true
              condition_icons: true
              round_temp: false
              type: daily
              number_of_forecasts: '0'
            units:
              pressure: ''
              speed: ''
      - type: grid
        columns: 1
        cards:
          - type: custom:apexcharts-card
            graph_span: 24h
            header:
              show: true
              title: Rain Rates
            series:
              - entity: sensor.gw2000b_daily_rain_rate_piezo
                name: Daily
                stroke_width: 2
                curve: smooth
              - entity: sensor.gw2000b_hourly_rain_rate_piezo
                name: Hourly
                stroke_width: 2
                curve: smooth
              - entity: sensor.gw2000b_rain_rate_piezo
                name: Current
                stroke_width: 2
                curve: smooth
    badges: []
title: Weather Station Dashboard
views:
  - title: Weather Overview
    path: weather_overview
    icon: mdi:weather-cloudy
    type: sections
    sections:
      - type: grid
        columns: 3
        cards:
          - type: custom:mushroom-template-card
            primary: '{{ states(''sensor.gw2000b_outdoor_temperature'') }}°C'
            secondary: Feels like {{ states('sensor.gw2000b_feels_like_temperature') }}°C
            icon: mdi:thermometer
            layout: vertical
            fill_container: true
            tap_action:
              action: more-info
              entity: sensor.gw2000b_outdoor_temperature
            icon_color: >-
              {% set temp = states('sensor.gw2000b_outdoor_temperature') | float
              %} {% if temp > 25 %}red {% elif temp > 20 %}orange {% elif temp >
              15 %}yellow {% elif temp > 10 %}green {% else %}blue{% endif %}
          - type: custom:mushroom-template-card
            primary: '{{ states(''sensor.gw2000b_humidity'') }}%'
            secondary: Dewpoint {{ states('sensor.gw2000b_dewpoint') }}°C
            icon: mdi:water-percent
            layout: vertical
            fill_container: true
            tap_action:
              action: more-info
              entity: sensor.gw2000b_humidity
            icon_color: >-
              {% set humidity = states('sensor.gw2000b_humidity') | float %} {%
              if humidity > 85 %}red {% elif humidity > 70 %}orange {% elif
              humidity > 60 %}yellow {% else %}green{% endif %}
          - type: custom:mushroom-template-card
            primary: '{{ states(''sensor.gw2000b_rain_rate_piezo'') }} mm/hr'
            secondary: |
              {% if states('sensor.gw2000b_rain_rate_piezo') | float > 0 %}
                Rain Detected!
              {% else %}
                No Rain
              {% endif %}
            icon: mdi:weather-rainy
            layout: vertical
            fill_container: true
            icon_color: |-
              {% if states('sensor.gw2000b_rain_rate_piezo') | float > 0 %}
                red
              {% else %}
                green
              {% endif %}
            entity: sensor.gw2000b_rain_rate_piezo
            tap_action:
              action: more-info
          - type: custom:mushroom-template-card
            primary: Pressure Trends
            secondary: >
              {{ states('sensor.air_pressure_trend') }} | {{
              states('sensor.pressure_trend') }}
            icon: mdi:trending-up
            layout: vertical
            fill_container: true
            icon_color: ''
            grid_options:
              columns: 6
              rows: 2
          - type: picture-entity
            entity: camera.backyard
            name: Backyard Live
            show_state: true
            show_name: true
            camera_view: auto
      - type: grid
        columns: 2
        cards:
          - type: custom:mini-graph-card
            entities:
              - entity: sensor.gw2000b_outdoor_temperature
                name: Temperature
                color: '#e74c3c'
              - entity: sensor.gw2000b_feels_like_temperature
                name: Feels Like
                color: '#3498db'
            hours_to_show: 24
            points_per_hour: 4
            line_width: 2
            hour24: true
            show:
              labels: true
              name: true
              legend: true
          - type: custom:apexcharts-card
            graph_span: 24h
            header:
              show: true
              title: Pressure Trends
            series:
              - entity: sensor.gw2000b_absolute_pressure
                name: Absolute
                stroke_width: 2
                curve: smooth
              - entity: sensor.gw2000b_relative_pressure
                name: Relative
                stroke_width: 2
                curve: smooth
            yaxis:
              - min: 950
                max: 1050
                apex_config:
                  tickAmount: 5
      - type: grid
        columns: 2
        cards:
          - type: iframe
            url: >-
              https://api.weather.com/v2/maps/dynamic?geocode=51.70,-2.50&h=320&w=568&lod=9&product=twcRadarHcMosaic&map=light&format=jpg&language=en-GB&apiKey=71f92ea9dd2f4790b92ea9dd2f779061&a=0
            aspect_ratio: 50%
            title: Current Radar
          - type: vertical-stack
            cards:
              - type: vertical-stack
                cards:
                  - type: custom:mushroom-template-card
                    primary: >-
                      {{ state_attr('binary_sensor.meteoalarm', 'headline') or
                      'No Active Alerts' }}
                    secondary: >-
                      {% set desc = state_attr('binary_sensor.meteoalarm',
                      'description') %} {% if desc %}
                        {{ desc }}
                      {% else %}
                        Currently there are no weather alerts for your area
                      {% endif %}
                    icon: |-
                      {% if is_state('binary_sensor.meteoalarm', 'on') %}
                        {% set awareness = state_attr('binary_sensor.meteoalarm', 'awareness_type') | lower %}
                        {% if awareness == 'wind' %}mdi:weather-windy
                        {% elif awareness == 'snow_ice' %}mdi:weather-snowy-heavy
                        {% elif awareness == 'thunderstorms' %}mdi:weather-lightning
                        {% elif awareness == 'fog' %}mdi:weather-fog
                        {% elif awareness == 'high_temperature' %}mdi:thermometer-high
                        {% elif awareness == 'low_temperature' %}mdi:thermometer-low
                        {% elif awareness == 'coastal_event' %}mdi:waves
                        {% elif awareness == 'forest_fire' %}mdi:fire-alert
                        {% elif awareness == 'avalanches' %}mdi:avalanche
                        {% elif awareness == 'rain' %}mdi:weather-pouring
                        {% elif awareness == 'flooding' %}mdi:home-flood
                        {% else %}mdi:alert-circle
                        {% endif %}
                      {% else %}
                        mdi:shield-check
                      {% endif %}
                    layout: vertical
                    fill_container: true
                    icon_color: |-
                      {% if is_state('binary_sensor.meteoalarm', 'on') %}
                        {% set level = state_attr('binary_sensor.meteoalarm', 'awareness_level') | int %}
                        {% if level == 2 %}yellow
                        {% elif level == 3 %}orange
                        {% elif level == 4 %}red
                        {% else %}green
                        {% endif %}
                      {% else %}
                        green
                      {% endif %}
                    tap_action:
                      action: more-info
                    hold_action:
                      action: url
                      url_path: https://www.meteoalarm.eu
          - type: tile
            entity: sensor.lydney_weather_alert_weathercom
            grid_options:
              columns: full
          - type: tile
            entity: sensor.lydney_weather_insight_weathercom
            grid_options:
              columns: full
          - type: tile
            entity: weather.forecast_home
            grid_options:
              columns: full
      - type: grid
        columns: 2
        cards:
          - type: custom:mini-graph-card
            entities:
              - entity: sensor.gw2000b_wind_speed
                name: Wind Speed
                color: '#3498db'
              - entity: sensor.gw2000b_wind_gust
                name: Wind Gust
                color: '#e74c3c'
            hours_to_show: 12
            points_per_hour: 6
            line_width: 2
            hour24: true
            show:
              labels: true
              points: true
              legend: true
          - type: custom:mushroom-template-card
            primary: Wind Direction
            secondary: >-
              {% set direction = states('sensor.gw2000b_wind_direction') | float
              %} {% set compass = [
                {'min': 348.75, 'max': 360, 'dir': 'N'},
                {'min': 0, 'max': 11.25, 'dir': 'N'},
                {'min': 11.25, 'max': 33.75, 'dir': 'NNE'},
                {'min': 33.75, 'max': 56.25, 'dir': 'NE'},
                {'min': 56.25, 'max': 78.75, 'dir': 'ENE'},
                {'min': 78.75, 'max': 101.25, 'dir': 'E'},
                {'min': 101.25, 'max': 123.75, 'dir': 'ESE'},
                {'min': 123.75, 'max': 146.25, 'dir': 'SE'},
                {'min': 146.25, 'max': 168.75, 'dir': 'SSE'},
                {'min': 168.75, 'max': 191.25, 'dir': 'S'},
                {'min': 191.25, 'max': 213.75, 'dir': 'SSW'},
                {'min': 213.75, 'max': 236.25, 'dir': 'SW'},
                {'min': 236.25, 'max': 258.75, 'dir': 'WSW'},
                {'min': 258.75, 'max': 281.25, 'dir': 'W'},
                {'min': 281.25, 'max': 303.75, 'dir': 'WNW'},
                {'min': 303.75, 'max': 326.25, 'dir': 'NW'},
                {'min': 326.25, 'max': 348.75, 'dir': 'NNW'}
              ] %} {% set curr_dir = 'N' %} {% for item in compass %}
                {% if direction >= item.min and direction < item.max %}
                  {% set curr_dir = item.dir %}
                {% endif %}
              {% endfor %} {{ direction }}° {{ curr_dir }}
            icon: >
              {% set direction = states('sensor.gw2000b_wind_direction') | float
              %}

              {% set icon_map = {
                'N': 'arrow-up',
                'NNE': 'arrow-up',
                'NE': 'arrow-top-right',
                'ENE': 'arrow-top-right',
                'E': 'arrow-right',
                'ESE': 'arrow-right',
                'SE': 'arrow-bottom-right',
                'SSE': 'arrow-bottom-right',
                'S': 'arrow-down',
                'SSW': 'arrow-down',
                'SW': 'arrow-bottom-left',
                'WSW': 'arrow-bottom-left',
                'W': 'arrow-left',
                'WNW': 'arrow-left',
                'NW': 'arrow-top-left',
                'NNW': 'arrow-top-left'
              } %}

              {% set compass = [
                {'min': 348.75, 'max': 360, 'dir': 'N'},
                {'min': 0, 'max': 11.25, 'dir': 'N'},
                {'min': 11.25, 'max': 33.75, 'dir': 'NNE'},
                {'min': 33.75, 'max': 56.25, 'dir': 'NE'},
                {'min': 56.25, 'max': 78.75, 'dir': 'ENE'},
                {'min': 78.75, 'max': 101.25, 'dir': 'E'},
                {'min': 101.25, 'max': 123.75, 'dir': 'ESE'},
                {'min': 123.75, 'max': 146.25, 'dir': 'SE'},
                {'min': 146.25, 'max': 168.75, 'dir': 'SSE'},
                {'min': 168.75, 'max': 191.25, 'dir': 'S'},
                {'min': 191.25, 'max': 213.75, 'dir': 'SSW'},
                {'min': 213.75, 'max': 236.25, 'dir': 'SW'},
                {'min': 236.25, 'max': 258.75, 'dir': 'WSW'},
                {'min': 258.75, 'max': 281.25, 'dir': 'W'},
                {'min': 281.25, 'max': 303.75, 'dir': 'WNW'},
                {'min': 303.75, 'max': 326.25, 'dir': 'NW'},
                {'min': 326.25, 'max': 348.75, 'dir': 'NNW'}
              ] %}

              {% set curr_dir = 'N' %}

              {% for item in compass %}
                {% if direction >= item.min and direction < item.max %}
                  {% set curr_dir = item.dir %}
                {% endif %}
              {% endfor %}

              mdi:{{ icon_map[curr_dir] }}
            layout: vertical
            fill_container: true
            icon_color: >
              {% set direction = states('sensor.gw2000b_wind_direction') | float
              %}

              {% set colors = {
                'N': 'blue',
                'NNE': 'blue',
                'NE': 'purple',
                'ENE': 'purple',
                'E': 'red',
                'ESE': 'red',
                'SE': 'orange',
                'SSE': 'orange',
                'S': 'yellow',
                'SSW': 'yellow',
                'SW': 'green',
                'WSW': 'green',
                'W': 'cyan',
                'WNW': 'cyan',
                'NW': 'blue',
                'NNW': 'blue'
              } %}

              {% set compass = [
                {'min': 348.75, 'max': 360, 'dir': 'N'},
                {'min': 0, 'max': 11.25, 'dir': 'N'},
                {'min': 11.25, 'max': 33.75, 'dir': 'NNE'},
                {'min': 33.75, 'max': 56.25, 'dir': 'NE'},
                {'min': 56.25, 'max': 78.75, 'dir': 'ENE'},
                {'min': 78.75, 'max': 101.25, 'dir': 'E'},
                {'min': 101.25, 'max': 123.75, 'dir': 'ESE'},
                {'min': 123.75, 'max': 146.25, 'dir': 'SE'},
                {'min': 146.25, 'max': 168.75, 'dir': 'SSE'},
                {'min': 168.75, 'max': 191.25, 'dir': 'S'},
                {'min': 191.25, 'max': 213.75, 'dir': 'SSW'},
                {'min': 213.75, 'max': 236.25, 'dir': 'SW'},
                {'min': 236.25, 'max': 258.75, 'dir': 'WSW'},
                {'min': 258.75, 'max': 281.25, 'dir': 'W'},
                {'min': 281.25, 'max': 303.75, 'dir': 'WNW'},
                {'min': 303.75, 'max': 326.25, 'dir': 'NW'},
                {'min': 326.25, 'max': 348.75, 'dir': 'NNW'}
              ] %}

              {% set curr_dir = 'N' %}

              {% for item in compass %}
                {% if direction >= item.min and direction < item.max %}
                  {% set curr_dir = item.dir %}
                {% endif %}
              {% endfor %}

              {{ colors[curr_dir] }}
            tap_action:
              action: more-info
            grid_options:
              columns: full
              rows: 2
          - type: gauge
            entity: sensor.gw2000b_wh90_battery
            max: 5
            min: 0
            needle: false
            grid_options:
              columns: 6
              rows: auto
          - type: gauge
            entity: sensor.gw2000b_wh90_capacitor
            max: 5
            min: 0
            grid_options:
              columns: 6
              rows: auto
      - type: grid
        columns: 2
        cards:
          - type: custom:apexcharts-card
            graph_span: 12h
            header:
              show: true
              title: Solar Radiation & UV
            series:
              - entity: sensor.gw2000b_solar_radiation
                name: Solar Radiation
                stroke_width: 2
                curve: smooth
              - entity: sensor.gw2000b_uv_index
                name: UV Index
                stroke_width: 2
                curve: smooth
                group_by:
                  func: avg
                  duration: 10min
              - entity: sensor.gw2000b_solar_lux
                name: Solar Lux
                stroke_width: 2
                curve: smooth
                group_by:
                  func: avg
                  duration: 10min
          - type: custom:weather-chart-card
            entity: weather.forecast_home
            show_main: true
            show_temperature: true
            show_current_condition: true
            show_attributes: true
            show_time: false
            show_time_seconds: false
            show_day: false
            show_date: false
            show_humidity: true
            show_pressure: true
            show_wind_direction: true
            show_wind_speed: true
            show_sun: true
            show_feels_like: false
            use_12hour_format: false
            icons_size: 25
            animated_icons: false
            icon_style: style1
            forecast:
              precipitation_type: rainfall
              show_probability: false
              labels_font_size: '11'
              precip_bar_size: '100'
              style: style1
              show_wind_forecast: true
              condition_icons: true
              round_temp: false
              type: daily
              number_of_forecasts: '0'
            units:
              pressure: ''
              speed: ''
      - type: grid
        columns: 1
        cards:
          - type: custom:apexcharts-card
            graph_span: 24h
            header:
              show: true
              title: Rain Rates
            series:
              - entity: sensor.gw2000b_daily_rain_rate_piezo
                name: Daily
                stroke_width: 2
                curve: smooth
              - entity: sensor.gw2000b_hourly_rain_rate_piezo
                name: Hourly
                stroke_width: 2
                curve: smooth
              - entity: sensor.gw2000b_rain_rate_piezo
                name: Current
                stroke_width: 2
                curve: smooth
    badges: []
