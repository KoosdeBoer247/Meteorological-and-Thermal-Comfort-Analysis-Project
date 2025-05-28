# Meteorological-and-Thermal-Comfort-Analysis-Project
This project provides a command-line tool for comprehensive meteorological and thermal comfort analysis. It leverages the OpenWeather API to fetch current weather data and utilizes the `pythermalcomfort` library for core thermal modeling.

Key functionalities include:
- Fetching current weather data for a specified city using the OpenWeather API, including:
    - 5-day/3-hour forecast data.
    - Current Air Quality Index (AQI).
- **Forecast-based Analysis:** Calculations are based on user-selected day and 3-hour time slot from the forecast.
- **Weather Data Interpolation:** Interpolates 3-hourly forecast data to finer intervals (e.g., 5-minute) for detailed time-series analysis of environmental conditions and UTCI.
- **Advanced Solar Radiation Modelling:** Utilizes `pvlib` to calculate Global Horizontal Irradiance (GHI), Direct Normal Irradiance (DNI), and Diffuse Horizontal Irradiance (DHI), considering time, location, cloud cover, and AQI (via Linke Turbidity).
- **Refined Mean Radiant Temperature (MRT):** MRT calculation now leverages the detailed solar radiation data for greater accuracy in outdoor environments. A comparative MRT from a user-provided globe temperature is also available.
- **Selectable Human Parameters:** Users can choose from predefined lists of activities (determining MET and posture) and clothing ensembles (determining CLO value) using simple IDs for single-run scenarios.
- Calculating standard thermal comfort indices like UTCI (Universal Thermal Climate Index) and WBGT (Wet Bulb Globe Temperature - if globe temperature is provided).
- Estimating human water loss rates (single point) and cumulative water loss over time using the JOS-3 physiological model.
- **Time-Series Physiological Simulation:** Simulates human physiological response (rectal temperature, water loss) over user-defined durations using the JOS-3 model, based on the conditions of the selected forecast slot.
- **Comprehensive Risk Assessment:**
    - Classifies risk based on UTCI, WBGT (if available), predicted rectal temperature from JOS-3, and AQI.
    - Determines overall risk progression over time, where the most severe instantaneous risk factor is dominant.
- Performing Monte Carlo simulations on key human parameters (metabolic rate, clothing insulation) to assess their impact on:
    - Scalar UTCI values.
    - Scalar JOS-3 water loss rates.
    - JOS-3 time-series outputs (rectal temperature, cumulative water loss), using initial forecast slot conditions as the baseline environment.
- **Graphical Output:** Generates plots for (if enabled via `--generate_plots`):
    - Rectal temperature over time (mean and uncertainty bands from Monte Carlo JOS-3 simulations).
    - Cumulative water consumption over time (mean and uncertainty bands from Monte Carlo JOS-3 simulations).
    - Overall risk level progression over the JOS-3 simulation period (based on a single JOS-3 time-series run, integrating thermal and AQI risks).
- Calculating Urban Heat Island (UHI) intensity based on Oke's empirical formula if city population is provided.

## Project Structure

```
meteo_comfort_project/
├── src/                  # Source code for calculations and data handling
│   ├── __init__.py
│   ├── data_fetcher.py     # OpenWeather API communication, timezone
│   ├── thermal_models.py   # pythermalcomfort models (UTCI, WBGT, MRT, JOS-3)
│   ├── uhi_calculator.py   # Urban Heat Island calculation
│   ├── simulation.py       # Monte Carlo simulation logic
│   ├── solar_calculator.py # Solar radiation calculations using pvlib
│   ├── risk_assessment.py  # Individual and overall risk classification
│   ├── plotting.py         # Plot generation using matplotlib
│   ├── utils.py            # Utility functions (e.g., weather interpolation)
│   └── human_param_options.py # Predefined MET/CLO/Posture options
├── tests/                # Unit tests
│   ├── __init__.py
│   ├── test_data_fetcher.py
│   ├── test_thermal_models.py
│   ├── test_uhi_calculator.py
│   ├── test_simulation.py
│   ├── test_solar_calculator.py
│   ├── test_risk_assessment.py
│   ├── test_plotting.py
│   └── test_utils.py
├── main.py               # Main command-line application script
├── README.md             # This file
└── requirements.txt      # Python dependencies
```

## Setup Instructions

**1. Clone the Repository (if applicable)**
If you have downloaded this project as a zip file, extract it. If you are using Git:
```bash
git clone <repository_url>
cd meteo_comfort_project
```

**2. Install Dependencies**
Navigate to the project root directory (`meteo_comfort_project`) and run:
```bash
pip install -r requirements.txt
```

**3. Set API Key**
This project requires an API key from OpenWeather (available at [https://openweathermap.org/api](https://openweathermap.org/api)).
It's recommended to set your API key as an environment variable for security and convenience:
```bash
export OPENWEATHER_API_KEY="YOUR_API_KEY"
```
Replace `"YOUR_API_KEY"` with your actual key.
Alternatively, you can pass the API key directly using the `--api_key` command-line argument when running `main.py`, but this is less secure, especially in shared environments or scripts.

## How to Run

The main script `main.py` is used to run all calculations and simulations.

To see all available options and their descriptions, run:
```bash
python main.py --help
```

Key arguments for new features:
*   `--forecast_day_offset OFFSET`: Choose which day from the 5-day forecast to use (0 for today, 1 for tomorrow, etc.).
*   `--forecast_time_index INDEX`: Select a specific 3-hour time slot within the chosen forecast day (0-7, e.g., 0 for 00:00-03:00, 4 for 12:00-15:00).
*   `--list_options`: Display available activity and clothing options with their IDs.
*   `--activity_id ID`: Select an activity by its ID to set MET and posture for single-run calculations.
*   `--clothing_id ID`: Select a clothing ensemble by its ID to set CLO value for single-run calculations.
*   `--generate_plots`: Enables the generation of all supported plots. Plots are saved in the directory specified by `--plot_output_dir`.
*   `--plot_output_dir PLOT_DIR`: Specifies the directory where generated plots will be saved. Defaults to `plots/` in the project root.
*   `--jos3_output_interval MINUTES`: Sets the interval in minutes for data points in the JOS-3 time-series simulation and for weather interpolation. Default is 5 minutes.
*   `--duration MINUTES`: Specifies the total duration in minutes for the JOS-3 physiological simulation.

**Example Commands:**

*   List available predefined activities and clothing:
    ```bash
    python main.py --list_options
    ```

*   Run analysis for Berlin, tomorrow, around noon (12:00-15:00 slot), for "Wandelen, 4.0 km/u" (ID 13) with "Lange broek en T-shirt" (ID 3), for a 180-minute duration, and generate plots:
    ```bash
    # Ensure OPENWEATHER_API_KEY is set or use --api_key YOUR_KEY
    python main.py --city "Berlin" --forecast_day_offset 1 --forecast_time_index 4 --activity_id 13 --clothing_id 3 --duration 180 --generate_plots
    ```

*   Run analysis for Phoenix, today, first forecast slot, for "Hardlopen, 9.6 km/u" (ID 2) with "Korte broek en tanktop" (ID 2), including Monte Carlo simulations and plots:
    ```bash
    # (Ensure OPENWEATHER_API_KEY is set or use --api_key YOUR_KEY)
    python main.py --city "Phoenix" --activity_id 2 --clothing_id 2 --duration 120 --jos3_output_interval 10 --run_simulation --num_simulations 100 --generate_plots --plot_output_dir "output_phoenix_run"
    ```

*   Generate only the JOS-3 based risk progression plot for Miami, using default activity/clothing for a 90-minute duration:
    ```bash
    python main.py --city "Miami" --duration 90 --jos3_output_interval 5 --generate_plots
    ```

## Output Explanation

The script provides output in several sections:

1.  **User Parameters for Single-Run Calculations:** Confirms the MET, CLO, and posture values derived from `--activity_id` and `--clothing_id` (or defaults).
2.  **Conditions for Selected Forecast Slot:** Displays the weather data (temperature, humidity, wind, cloud cover) for the chosen 3-hour forecast slot, including its local date and time.
3.  **Solar Radiation & Initial MRT Calculation:** Shows calculated GHI, DNI, solar zenith, and the resulting initial Mean Radiant Temperature (`initial_tr_solar`) used for subsequent calculations.
4.  **Interpolated Weather Data Summary (if applicable):** If weather interpolation is performed (typically for time-stepped UTCI or detailed environmental risk plotting), a message indicates success and the number of steps.
5.  **Time-Stepped UTCI & Environmental Risk (if interpolated data available):** Provides a summary (min, max, avg) of UTCI calculated over the interpolated time steps.
6.  **Scalar & JOS-3 Time-Series Calculations (based on initial slot conditions):**
    *   **UTCI (Initial Slot):** UTCI value and its risk category for the precise start of the JOS-3 simulation period.
    *   **WBGT (if `--globe_temp` provided):** WBGT value and risk for the initial conditions.
    *   **JOS-3 Human Water Loss (Scalar):** Predicted water loss *rates* (e.g., g/h) at the end of the specified `--duration`, assuming constant initial environmental conditions.
    *   **UHI (if `--population` provided):** Estimated Urban Heat Island intensity.
    *   **Risk Progression Plot (if `--generate_plots`):**
        *   Summary of initial and final overall risk based on the JOS-3 physiological simulation (combining initial UTCI risk, WBGT risk if available, time-varying rectal temperature risk, and constant AQI risk).
        *   Saves `risk_progression_jos3_plot.png` to the plot output directory.
7.  **Monte Carlo Simulation Results (if `--run_simulation` is used):**
    *   **Scalar Outputs:** Summarizes simulation results for UTCI and JOS-3 water loss rates, providing statistics like mean and median based on variations in user MET and CLO (defined by `--*_sim_mean`/`_std` args). These simulations use the initial selected forecast slot's conditions as their baseline environment.
    *   **Time-Series Outputs & Plots (if `--generate_plots` is also used):**
        *   `rectal_temp_simulation_plot.png`: Shows the mean predicted rectal temperature over time from JOS-3 MC simulations, with an uncertainty band.
        *   `water_loss_simulation_plot.png`: Shows the mean cumulative water consumption over time from JOS-3 MC simulations, with an uncertainty band.
        *   Plots are saved in the directory specified by `--plot_output_dir`.

## Running Tests

To run the unit tests, navigate to the project root directory and execute:
```bash
python -m unittest discover tests
```
This command will automatically find and run all tests within the `tests` directory.

## Key Dependencies
The project relies on the following key Python libraries:
-   `requests`: For making HTTP requests to the OpenWeather API.
-   `numpy`: For numerical operations, especially in the Monte Carlo simulation.
-   `pandas`: For data manipulation, particularly for time series and date parsing.
-   `pvlib`: For detailed solar radiation calculations.
-   `timezonefinder`: For determining local timezones from coordinates.
-   `pythermalcomfort`: For the core thermal comfort and physiological models.
-   `matplotlib`: For generating plots.
```
```
