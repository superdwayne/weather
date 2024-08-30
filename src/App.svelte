<script>
  let city = ''; // City input
  let weatherData = null; // Store weather data
  let loading = false; // Loading state
  let error = ''; // Error state

  async function fetchWeather() {
    if (!city) return;

    loading = true;
    error = '';
    weatherData = null;

    try {
      const apiKey = '0128f326def470b7fe59e5815edbc56b'; // Your OpenWeatherMap API key
      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=metric`
      );

      if (!response.ok) {
        throw new Error('City not found');
      }

      weatherData = await response.json();
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

  function getWeatherIconPath() {
    if (!weatherData) return '';

    const weatherMain = weatherData.weather[0].main.toLowerCase();

    // Map the weather condition to the SVG icon filename
    switch (weatherMain) {
      case 'clear':
        return '/src/assets/all/clear-day.svg';
      case 'rain':
        return '/src/assets/all/rain.svg';
      case 'drizzle':
        return '/src/assets/all/drizzle.svg';
      case 'clouds':
        return '/src/assets/all/cloudy.svg';
      case 'thunderstorm':
        return '/src/assets/all/thunderstorms.svg';
      case 'snow':
        return '/src/assets/all/snow.svg';
      case 'mist':
        return '/src/assets/all/fog.svg';
      case 'smoke':
        return '/src/assets/all/smoke.svg';
      case 'haze':
        return '/src/assets/all/haze.svg';
      case 'dust':
        return '/src/assets/all/dust.svg';
      default:
        return '/src/assets/all/not-available.svg'; // Generic icon for unknown cases
    }
  }
</script>

<div class="weather-app">
  <!-- Input for city name -->
  <input
    type="text"
    bind:value={city}
    placeholder="Enter city name"
    on:keyup="{(e) => e.key === 'Enter' && fetchWeather()}"
  />
  <button on:click={fetchWeather}>Get Weather</button>

  <!-- Display loading state, errors, and weather data -->
  {#if loading}
    <p>Loading...</p>
  {/if}

  {#if error}
    <p class="error">{error}</p>
  {/if}

  {#if weatherData}
    <div class="weather-info">
      <h2>Weather in {weatherData.name}</h2>
      <p>Temperature: {weatherData.main.temp}°C</p>
      <p>Humidity: {weatherData.main.humidity}%</p>
      <p>Condition: {weatherData.weather[0].description}</p>

      <!-- Weather Icon Display -->
      <div class="weather-icon">
        <img src={getWeatherIconPath()} alt="Weather Icon" />
      </div>
    </div>
  {/if}
</div>

<style>
  .weather-app {
    max-width: 400px;
    margin: 0 auto;
    text-align: center;
  }

  .weather-info {
    margin-top: 20px;
  }

  .weather-icon {
    font-size: 48px; /* Adjust icon size */
    margin-top: 20px;
  }

  .error {
    color: red;
  }
</style>
