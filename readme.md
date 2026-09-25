![WebGL Black Hole Preview](./public/assets/preview.png)

# WebGL Black Hole

 A mesmerizing WebGL simulation of a black hole — where physics meets art, and code becomes cosmos.

---

## Table of Contents

- [The Idea](#-the-idea)
- [What You'll See](#-what-youll-see)
- [Architecture](#-architecture)
- [Shaders & Visual Effects](#-shaders--visual-effects)
- [Debug Mode](#-debug-mode)
- [Getting Started](#-getting-started)
- [Project Structure](#-project-structure)
- [Contributing](#-contributing)
- [License](#-license)
- [Credits](#-credits)

---

## The Idea

This project began as a tribute to one of the most enigmatic objects in the universe — the black hole. Inspired by [Bruno Simon's iconic WebGL Black Hole](https://github.com/brunosimon/webgl-black-hole), it reimagines the experience with custom shaders, multi-pass rendering, and a deeply crafted particle system.

The goal is simple yet ambitious: **make someone stop scrolling and stare at the screen**.

---

## What You'll See

When you load the simulation, you're greeted by a scene composed of multiple interconnected layers:

1. **Starfield** — 50,000 uniquely colored stars scattered across a massive sphere, each with its own size and hue
2. **Accretion Disc** — A glowing, animated ring of gas swirling around the event horizon, driven by procedural noise
3. **Particle Cloud** — 50,000 luminous particles orbiting the singularity, fading from inner amber to outer crimson
4. **Gravitational Distortion** — A real-time RGB shift effect that warps the starfield around the black hole's center

All rendered through a **three-pass compositing pipeline**: space scene → distortion map → final screen blend.

```
   ┌─────────────┐
   │   Space     │  ──→  Starfield + Accretion Disc
   │  Scene      │
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │ Distortion  │  ──→  Radial mask + active distortion
   │  Pass       │
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │   Final     │  ──→  RGB shift compositing + overlay
   │  Composite  │
   └─────────────┘
```

---

## Architecture

The project follows a clean, modular class-based architecture built on Three.js. Every system is decoupled and single-responsibility.

```
Experience (singleton)
├── Time — frame-timing & play/pause control
├── Sizes — viewport tracking & resize events
├── Debug — conditional UI & stats panel (#debug)
├── Camera — dual-mode (default / orbit) with smooth blending
├── Renderer — multi-render-target composition pipeline
├── Resources — async asset loading (GLTF, DRACO, textures)
├── World
│   ├── Stars — 50k particles in a spherical shell
│   ├── BlackHole
│   │   ├── Disc — noise-animated accretion ring
│   │   ├── Particles — orbiting luminance field
│   │   └── Distortion — mask + active displacement
│   └── Spaceship — mouse-following capsule with cupola
└── Noises — procedural Perlin noise texture generation
```

The `Experience` class enforces a **singleton pattern**, ensuring only one instance exists throughout the application lifecycle. Each subsystem subscribes to shared utilities and communicates through well-defined events.

---

## Shaders & Visual Effects

Every visual element is powered by **custom GLSL 3.0 shaders** — no built-in Three.js materials are used for the core scene.

### Shader Inventory

| Shader | Purpose | Key Technique |
|---|---|---|
| `blackHoleDisc` | Accretion ring rendering | Procedural noise animation with color blending |
| `blackHoleParticles` | Orbiting particle field | Polar coordinate animation with size attenuation |
| `starsParticles` | Background starfield | Spherical distribution with per-particle color |
| `noises` | Noise texture generation | 3D periodic Perlin noise |
| `blackHoleDistortionActive` | Distortion intensity map | Radial gradient from center |
| `blackHoleDistortionMask` | Distortion shape mask | Smooth radial falloff |
| `final` | Screen-space compositing | RGB chromatic shift toward black hole |

### The Final Composite

The `FinalMaterial` is where the magic converges. It samples both the space and distortion textures, computes a displacement vector pointing toward the black hole's screen position, and applies a **chromatic aberration** effect based on angular offset — creating the illusion of gravitational lensing.

```glsl
vec2 towardCenter = vUv - uBlackHolePosition;
towardCenter *= - distortionIntensity * 2.0;
vec2 distoredUv = vUv + towardCenter;
vec3 outColor = getRGBShiftedColor(uSpaceTexture, distoredUv, uRGBShiftRadius);
```

---

## Debug Mode

The project ships with a fully featured debug environment. Activate it by appending `#debug` to the URL:

```
http://localhost:5173/#debug
```

Once enabled, you gain access to:

- **lil-gui** panel with folders for `blackhole`, `camera`, `renderer`, and `spaceship/directionalLight`
- **stats.js** performance overlay with GPU timing via `EXT_disjoint_timer_query_webgl2`
- **OrbitControls** for free camera navigation
- Real-time parameter tuning — colors, intensities, shadows, tone mapping

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) ≥ 16
- [npm](https://www.npmjs.com/) or [yarn](https://yarnpkg.com/)

### Installation

```bash
# Clone the repository
git clone https://github.com/sebastianvasquezechavarria1234/black-hole.git
cd black-hole

# Install dependencies
npm install
```

### Development

```bash
# Start the dev server
npm run dev
```

The application will be available at `http://localhost:5173`.

### Build

```bash
# Create a production build
npm run build
```

Output is generated in the `dist/` directory.

### Preview

```bash
# Preview the production build locally
npm run preview
```

### Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start Vite dev server with HMR |
| `npm run build` | Build for production |
| `npm run preview` | Preview production build |

---

## Project Structure

```
black-hole/
├── sources/
│   ├── index.js              # Application entry point
│   ├── style.css             # Global styles & layout
│   ├── index.html            # HTML shell with OG meta tags
│   ├── Experience/
│   │   ├── Experience.js     # Singleton orchestrator
│   │   ├── World.js          # Scene composition
│   │   ├── Camera.js         # Dual-mode camera system
│   │   ├── Renderer.js       # WebGL renderer & composition
│   │   ├── Stars.js          # Starfield particle system
│   │   ├── BlackHole.js      # Black hole core (disc, particles, distortion)
│   │   ├── Spaceship.js      # Mouse-following spacecraft
│   │   ├── Cupola.js         # 3D cupola model wrapper
│   │   ├── Noises.js         # Procedural noise generator
│   │   ├── Resources.js      # Async asset manager
│   │   ├── assets.js         # Asset group definitions
│   │   ├── Materials/        # Custom shader material wrappers
│   │   │   ├── BlackHoleDiscMaterial.js
│   │   │   ├── BlackHoleParticlesMaterial.js
│   │   │   ├── StarsParticlesMaterial.js
│   │   │   ├── FinalMaterial.js
│   │   │   ├── NoisesMaterial.js
│   │   │   ├── BlackHoleDistortionActiveMaterial.js
│   │   │   └── BlackHoleDistortionMaskMaterial.js
│   │   ├── shaders/          # GLSL source files
│   │   │   ├── blackHoleDisc/
│   │   │   ├── blackHoleParticles/
│   │   │   ├── starsParticles/
│   │   │   ├── noises/
│   │   │   ├── blackHoleDistortionActive/
│   │   │   ├── blackHoleDistortionMask/
│   │   │   ├── final/
│   │   │   └── partials/     # Shared Perlin noise implementations
│   │   ├── Debug/            # Debug UI & stats
│   │   └── Utils/            # Time, Sizes, EventEmitter, Loader
├── public/
│   ├── assets/               # Models, textures, images
│   │   ├── models/cupola.glb
│   │   └── lenna.png
│   ├── draco/                # Draco compression binaries
│   ├── basis/                # Basis texture transcoder
│   └── social/               # Open Graph preview images
├── resources/                # Preview media (images, videos)
├── vite.config.js            # Vite configuration with GLSL plugin
├── package.json
└── LICENSE
```

---

## Dependencies

| Package | Version | Role |
|---|---|---|
| `three` | ^0.141.0 | 3D rendering engine |
| `vite` | ^2.9.12 | Build tool & dev server |
| `vite-plugin-glsl` | ^0.1.2 | GLSL shader importing |
| `lil-gui` | ^0.16.1 | Debug UI panel |
| `stats.js` | ^0.17.0 | Performance monitoring |
| `stylus` | ^0.58.1 | CSS preprocessor |

---

## Contributing

Contributions are the lifeblood of this project. If you spot a bug, have a feature idea, or want to improve the shaders — open an issue or submit a pull request.

### Guidelines

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** with clarity (`git commit -m 'Add some amazing feature'`)
4. **Push** to your branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request

---

## License

This project is licensed under the [MIT License](LICENSE).

> **Note:** This project is based on original work by [Bruno Simon](https://github.com/brunosimon) from the [webgl-black-hole](https://github.com/brunosimon/webgl-black-hole) repository.

---

## Credits

- **Bruno Simon** — Original [webgl-black-hole](https://github.com/brunosimon/webgl-black-hole) project that inspired this work
- **Stefan Gustavson** — Classic Perlin noise implementations used in the shader partial functions
- **Three.js Community** — For the incredible ecosystem and documentation

---

<p align="center">
Made with ❤️ by <a href="https://sebas-dev.vercel.app/" target="_blank" rel="noopener noreferrer">Sebastián V</a>

</p>
