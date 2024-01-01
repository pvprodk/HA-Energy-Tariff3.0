# HA-Energy-Tariff3.0
A guide to setup Home Assistant Energy Dashboard to show electricity purchase based on Tariff 3.0 

## Step 1 - Create a Template sensor:
The purpose of the template sensor is just to give a value "lavlast", "hojlast" or "spidslast" based on the time of day.

Make sure to place the code in your templates section and make the appropriate indentation.
```
- sensor:
    # Lavlast (00-06)
    # Højlast (06-17) & (21-00)
    # Spidslast (17-21)
    - name: "Tariff 3.0"
      unique_id: 541d7af3-7518-42b4-ad78-7fba858c7770
      state: "{{ 'spidslast' if today_at('17:00') < now() < today_at('21:00') else ('lavlast' if today_at('00:00') < now() < today_at('06:00') else 'hojlast') }}"
```


## Step 2 - Create a Utility Helper:
Creating this Utility Helper will provide a “base” sensor (which controls which tariff is currently in use), and one sensor for each tariff (which counts the amount of energy used on each tariff)

Make sure to choose the correct sensor from your power-meter or similiar (in this example "Energy Imported" from a Slimmelezer connected to Landis+Gyr Electricity meter), the name will probably differ from this example

Make sure to type in the helper name and the three tariff names exactly.

![Skærmbillede 2024-01-01 214147](https://github.com/pvprodk/HA-Energy-Tariff3.0/assets/79306514/266917ed-0e5f-4036-bdaf-ade0e886164f)


## Step 3 - Create an automation
The purpose of this automation is to update the utility helper created in step 2, to the current tariff when the tariff changes

After creating the automation, run it manually the first time, to syncronize the Utility helper with the current tariff
```
alias: Set Electricity Tariff
description: ""
trigger:
  - platform: state
    entity_id: sensor.tariff_3_0
    not_to:
      - unavailable
      - unknown
action:
  - service: select.select_option
    target:
      entity_id: select.tariff_3_0_import
    data:
      option: "{{ trigger.to_state.state if 'to_state' in trigger else 'hojlast' }}"
mode: single
```

## Step 4 - Add the three newly created Utility helpers to the Energy Dashboard:
Do this for all 3 tariffs

I recommend using [EnergiDataService](https://github.com/MTrab/energidataservice) for the price information on the sensors

![image](https://github.com/pvprodk/HA-Energy-Tariff3.0/assets/79306514/7b2c34a2-8e73-407b-8b1f-bb74bfe7ca1c)

And then you get the information presented in your Energy Dashboard like this:
![image](https://github.com/pvprodk/HA-Energy-Tariff3.0/assets/79306514/3be37acf-aa04-4323-80de-82d1a96f7901)


