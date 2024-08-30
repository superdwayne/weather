<script>
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

  let city = ''; // City input
  let weatherData = null; // Store weather data
  let loading = false; // Loading state
  let error = ''; // Error state

  // To-Do List State
  let newTodo = ''; // New to-do input
  let todos = []; // List of to-dos

  // Stress level mapping
  const stressMapping = {
    'sofa': 8,
    'bed': 7,
    'refrigerator': 9,
    'tv': 6,
    'desk': 5,
  };

  // Store multiple 3D model URLs
  const modelUrls = [
    'https://storage.polycam.io/captures/beb37255-8ddd-4019-8675-03e5b446727e/raw.glb?t=1725021165339',
    'https://storage.polycam.io/captures/beb37255-8ddd-4019-8675-03e5b446727e/raw.glb?t=1725021165339'
    // Add more URLs here if needed
  ];
  let currentModelIndex = 0;

  // Load to-dos from local storage on component mount
  onMount(() => {
    const savedTodos = localStorage.getItem('todos');
    if (savedTodos) {
      todos = JSON.parse(savedTodos);
    }

    init3DViewer();
  });

  let scene, camera, renderer, controls, loader, currentModel;

  // Initialize Three.js 3D viewer
  function init3DViewer() {
    const canvas = document.getElementById('3d-canvas');
    renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
    renderer.setSize(window.innerWidth / 2, window.innerHeight / 2);
    renderer.setPixelRatio(window.devicePixelRatio);

    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera(75, window.innerWidth / 2 / (window.innerHeight / 2), 0.1, 1000);
    camera.position.set(0, 2, 5);

    // Add lighting to the scene
    const ambientLight = new THREE.AmbientLight(0x404040, 6); // Ambient light
    scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 1); // Directional light
    directionalLight.position.set(5, 5, 5);
    scene.add(directionalLight);

    loader = new GLTFLoader();
    loadModel(modelUrls[currentModelIndex]);

    // OrbitControls
    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.25;
    controls.enableZoom = true;

    function animate() {
      requestAnimationFrame(animate);
      controls.update();
      renderer.render(scene, camera);
    }
    animate();

    window.addEventListener('resize', onWindowResize);
  }

  function onWindowResize() {
    renderer.setSize(window.innerWidth / 2, window.innerHeight / 2);
    camera.aspect = window.innerWidth / 2 / (window.innerHeight / 2);
    camera.updateProjectionMatrix();
  }

  function loadModel(url) {
    if (currentModel) {
      scene.remove(currentModel);
    }

    loader.load(url, (gltf) => {
      currentModel = gltf.scene;
      currentModel.scale.set(1, 1, 1); // Adjust scale if needed
      currentModel.position.set(0, 0, 0); // Adjust position if needed
      scene.add(currentModel);
    }, undefined, (error) => {
      console.error('An error occurred while loading the model:', error);
    });
  }

  function nextModel() {
    currentModelIndex = (currentModelIndex + 1) % modelUrls.length;
    loadModel(modelUrls[currentModelIndex]);
  }

  function previousModel() {
    currentModelIndex = (currentModelIndex - 1 + modelUrls.length) % modelUrls.length;
    loadModel(modelUrls[currentModelIndex]);
  }

  // Fetch weather data
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

  // Get weather icon path
  function getWeatherIconPath() {
    if (!weatherData) return '';

    const weatherMain = weatherData.weather[0].main.toLowerCase();

    switch (weatherMain) {
      case 'clear':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/clear-day.svg';
      case 'rain':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/rain.svg';
      case 'drizzle':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/drizzle.svg';
      case 'clouds':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/cloudy.svg';
      case 'thunderstorm':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/thunderstorms.svg';
      case 'snow':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/snow.svg';
      case 'mist':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/fog.svg';
      case 'smoke':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/smoke.svg';
      case 'haze':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/haze.svg';
      case 'dust':
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/dust.svg';
      default:
        return 'https://bmcdn.nl/assets/weather-icons/v3.0/fill/svg/not-available.svg'; // Generic icon for unknown cases
    }
  }

  // Add a new to-do item
  function addTodo() {
    if (newTodo.trim() === '') return;

    const [text, keyword] = newTodo.split(/\s+/); // Split input into item and keyword
    const stressLevel = stressMapping[keyword?.toLowerCase()] || 1; // Default to 1 if keyword not recognized

    todos = [...todos, { text, stressLevel }];
    newTodo = ''; // Clear input field
    saveTodos();
  }

  // Remove a to-do item
  function removeTodo(index) {
    todos = todos.filter((_, i) => i !== index);
    saveTodos();
  }

  // Update stress level
  function updateStressLevel(index, level) {
    todos = todos.map((todo, i) =>
      i === index ? { ...todo, stressLevel: level } : todo
    );
    saveTodos();
  }

  // Save to-dos to local storage
  function saveTodos() {
    localStorage.setItem('todos', JSON.stringify(todos));
  }
</script>

<div class="dashboard">
  <!-- Weather Section -->
  <section class="weather-section">
    <h2>Weather</h2>
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
        <p><strong>City:</strong> {weatherData.name}</p>
        <p><strong>Temperature:</strong> {weatherData.main.temp}°C</p>
        <p><strong>Humidity:</strong> {weatherData.main.humidity}%</p>
        <p><strong>Condition:</strong> {weatherData.weather[0].description}</p>

        <!-- Weather Icon Display -->
        <div class="weather-icon">
          <img src={getWeatherIconPath()} alt="Weather Icon" />
        </div>
      </div>
    {/if}
  </section>

  <!-- To-Do List Section -->
  <section class="todo-section">
    <h2>To-Do List</h2>
    <input
      type="text"
      bind:value={newTodo}
      placeholder="Enter a new to-do and keyword (e.g., 'sofa')"
    />
    <button on:click={addTodo}>Add To-Do</button>

    <!-- Display to-do items -->
    <ul>
      {#each todos as todo, index}
        <li>
          <span>{todo.text}</span>
          <span>(Stress Level: {todo.stressLevel})</span>
          <button on:click={() => updateStressLevel(index, todo.stressLevel + 1)}>+</button>
          <button on:click={() => updateStressLevel(index, todo.stressLevel - 1)}>-</button>
          <button on:click={() => removeTodo(index)}>Remove</button>
        </li>
      {/each}
    </ul>
  </section>

  <!-- 3D Model Viewer Section -->
  <section class="model-section">
    <h2>3D Model Viewer</h2>
    <canvas id="3d-canvas"></canvas>
    <button on:click={previousModel}>Previous</button>
    <button on:click={nextModel}>Next</button>
  </section>
</div>

<style>
  .dashboard {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
  }

  .weather-section,
  .todo-section,
  .model-section {
    flex: 1;
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  }

  .weather-icon img {
    width: 50px;
    height: 50px;
  }

  #3d-canvas {
    width: 100%;
    height: 400px;
    border: 1px solid #ccc;
    border-radius: 8px;
  }

  .error {
    color: red;
  }
</style>
