# STEMLab: Code Walkthrough

This document explains the current implementation of [STEMLab](https://asmaareesha.github.io/StemLab/), based on the repository's `index.html`.

> **Implementation note:** The current version is a single-file static website. The HTML, CSS, and JavaScript are all in `index.html`; there is no separate `src/` folder or framework build step.

## 1. High-level architecture

STEMLab has four main parts:

1. **HTML structure** defines the navigation, experiment sections, canvases, controls, readout containers, and quiz container.
2. **CSS** defines the color system, typography, cards, responsive layout, focus states, and active/inactive sections.
3. **JavaScript simulation state and calculations** store the current settings and calculate projectile motion, orbital motion, and forces on a ramp.
4. **Canvas rendering and event handlers** draw the simulations and update them when users interact with controls.

The general interaction cycle is:

```text
User changes a slider or clicks a control
                  |
                  v
      JavaScript state is updated
                  |
                  v
   Simulation values are recalculated
                  |
                  v
     Canvas and readouts are redrawn
```

## 2. HTML structure

The document begins with standard HTML metadata in the `<head>`:

- `lang="en"` identifies the page language.
- The character encoding and viewport settings help text and layout display correctly on different devices.
- The title and description provide page metadata.
- Google Fonts supplies the preferred typefaces, with local fallback fonts defined in CSS.

The visible application sits inside `.app`.

### Navigation

The navigation buttons use `data-t` values to identify their target sections:

- `proj` — projectile motion
- `orb` — orbital motion
- `frc` — forces on a ramp
- `quiz` — knowledge-check quiz

Each experiment is a `<section>`. The active section has the `on` class; CSS hides the other sections.

### Canvas elements

The simulations use HTML `<canvas>` elements:

| Canvas ID | Purpose |
|---|---|
| `pc` | Projectile trajectory and velocity arrows |
| `pg` | Horizontal and vertical velocity components over time |
| `oc` | Satellite orbit and trajectory |
| `og` | Relative orbital energy error |
| `fc` | Ramp, block, and force arrows |
| `fg` | Acceleration versus ramp angle |

The `data-h` attribute supplies each canvas's intended display height. The `aria-label` attributes provide concise labels for the visual regions.

### Control and output containers

The JavaScript inserts slider controls into `pctl`, `octl`, and `fctl`. The `pkv`, `okv`, and `fkv` elements display calculated values. The quiz is rendered inside `qbox`.

## 3. CSS and visual design

The CSS is embedded in the `<style>` element.

### Color variables

The `:root` variables define reusable colors such as the background, text, card, border, pink accent, teal accent, and highlight color. Reusing variables keeps the visual language consistent and makes theme adjustments easier.

### Responsive layout

The `.lab` class uses CSS Grid to arrange the simulation and control panels. A media query switches the layout to one column on narrower screens.

### Interaction states

The CSS styles navigation buttons, hover states, focused buttons and inputs, quiz answers, and feedback panels. The `:focus-visible` rule adds a visible outline for keyboard navigation.

### Dark theme

A `prefers-color-scheme: dark` media query changes the color variables when the user's device prefers dark mode. The stylesheet also defines a `data-theme="dark"` override.

## 4. Shared JavaScript helpers

The script starts with a few reusable helpers.

### `$()`

```js
const $ = s => document.querySelector(s);
```

This short helper selects the first element matching a CSS selector. For example, `$('#pgo')` selects the projectile play/pause button.

### `css()`

```js
const css = n =>
  getComputedStyle(document.documentElement)
    .getPropertyValue(n).trim();
```

This reads a CSS custom property, allowing Canvas drawing colors to follow the current theme.

### `prep(id)`

`prep()` configures a canvas for drawing. It reads the canvas's displayed width and configured height, scales its internal pixel dimensions using `devicePixelRatio`, creates a 2D drawing context, and stores that context in the `cv` object.

This improves sharpness on high-density screens. The stored drawing context also keeps track of the canvas width and height used by the drawing functions.

### `arrow()` and `txt()`

- `arrow()` draws a line with an arrowhead and an optional label. It is used for velocity and force vectors.
- `txt()` draws text on a Canvas using the site's preferred font and muted text color by default.

### `slider()`

`slider()` generates labelled range inputs from configuration arrays. Each configuration specifies the input ID, label, minimum, maximum, step size, starting value, and unit.

When a slider changes, the helper updates its output label and calls a callback with the changed control's ID and numeric value.

### `setv()`

`setv()` sets a slider value and dispatches an `input` event. This allows quiz prompts to set experiment parameters through the same update logic used by normal user interaction.

## 5. Projectile motion

The projectile experiment uses the `P` object to store its parameters and current simulation state.

| Property | Meaning |
|---|---|
| `v` | Launch speed |
| `a` | Launch angle in degrees |
| `h` | Initial height |
| `g` | Gravitational acceleration |
| `k` | Air-drag coefficient |
| `t` | Current animation time |
| `pts` | Calculated simulation points |
| `dt` | Simulation time step |

### Updating parameters

The sliders control launch speed, angle, initial height, gravity, and air drag. Their callback updates the corresponding property in `P`, calls `simP()`, and resets the animation time.

### `simP()`: numerical calculation

The function converts the angle from degrees to radians and resolves the launch velocity into horizontal and vertical components:

\[
v_x = v\cos(\theta), \qquad v_y = v\sin(\theta)
\]

It then advances the simulated state in small time steps. Gravity acts vertically, while the simplified drag term opposes the velocity. The function records positions, velocity components, and time in `P.pts` until the projectile reaches ground level or reaches its safety time limit.

The integration uses intermediate acceleration estimates in a Runge–Kutta-style update. This is a numerical approximation of the motion, rather than an exact continuous solution.

The function also calculates analytical no-drag reference quantities and updates the numeric readouts, including range, peak height, time in the air, and the no-air-drag range.

**Important:** The simulated trajectory with drag and the analytical no-drag reference are different models. They should not be interpreted as the same result.

### `drawP()`: trajectory

`drawP()` clears the projectile canvas, chooses a scale that fits the current trajectory, draws grid lines and axes, and plots a dashed idealized no-drag path. It then draws the calculated trajectory up to the current animation time, marks the projectile's current position, and draws horizontal and vertical velocity arrows.

### `drawPG()`: velocity graph

`drawPG()` plots horizontal velocity (`vx`) and vertical velocity (`vy`) over time. A vertical marker indicates the current animation time.

### Scrubbing, pause, and replay

The `scrub()` helper converts pointer position on a canvas into a normalized value. The projectile graph uses it to move the animation to a selected point in time.

The play/pause button toggles `P.pause`; the replay button resets the time and resumes the animation. The code also checks `prefers-reduced-motion` to avoid automatically starting the projectile animation for users who request reduced motion.

## 6. Satellite orbit simulation

The `O` object stores the satellite's launch speed, position, velocity, numerical method, path history, energy-error history, and status.

### Initial conditions

`resetO()` sets the satellite's initial position at unit distance from the central body and gives it a tangential velocity determined by the launch-speed control. It clears the path and energy histories and records the initial specific orbital energy.

The orbit model uses normalized units. The on-page note states that the starting distance and gravitational pull are set to 1, circular speed is 1, and escape speed is approximately 1.41.

### `acc()`: gravitational acceleration

The `acc()` function calculates a central inverse-square acceleration. In vector form, the model is equivalent to:

\[
\vec{a} = -\frac{\vec{r}}{|\vec{r}|^3}
\]

This is a simplified two-dimensional model of a satellite moving under the gravity of a central body.

### `stepO(dt)`: numerical integration

`stepO()` supports two methods:

- **Leapfrog (`lf`)** updates velocity in half steps around the position update.
- **Euler (`eu`)** updates position using the current velocity and then updates velocity using the current acceleration.

The methods approximate the same underlying physical model but can produce different numerical errors. Comparing them helps demonstrate why a numerical method matters in a simulation.

### `drawO()`: orbit and diagnostics

`drawO()` advances the orbit in small increments, draws the trajectory and satellite, and displays a velocity vector. It calculates the current distance and speed, estimates orbital eccentricity and period where applicable, and classifies the outcome as a near-circular orbit, ellipse, escape, or collision with the central region.

The lower energy graph tracks relative change from the initial energy. In an ideal numerical solution, energy error would remain zero; in a numerical simulation, the graph helps show integration error.

## 7. Forces on a ramp

The `F` object stores the ramp angle, block mass, friction coefficient, and a simple animation-time variable.

### `fcalc()`: force calculation

The function converts the ramp angle to radians and calculates the relevant forces using \(g = 9.81\,\text{m/s}^2\).

Weight:

\[
W = mg
\]

Normal force for a block on a straight incline:

\[
N = mg\cos(\theta)
\]

Component of weight down the ramp:

\[
F_{\parallel} = mg\sin(\theta)
\]

The simplified friction limit used by the model is:

\[
F_f = \mu N
\]

The block is treated as sliding when the downhill component of weight exceeds the available friction. When sliding, the model calculates net force and acceleration:

\[
F_{\text{net}} = F_{\parallel} - F_f
\]

\[
a = \frac{F_{\text{net}}}{m}
\]

When the block is not sliding, the model sets its acceleration to zero and friction balances the downhill component, up to the friction limit.

### `drawF()`: force diagram

`drawF()` draws the ramp and block, then adds arrows for weight, normal force, friction, and net force when applicable. It updates the numeric force readouts and draws the acceleration-versus-angle graph.

The graph also marks the threshold angle predicted by the simplified condition \(\tan(\theta) = \mu\).

### Changing the angle from the graph

Pointer interaction with the acceleration graph converts the horizontal pointer position to an angle. `setv()` updates the ramp-angle slider, which then runs the same callback as a manual slider change.

## 8. Conceptual quiz

The quiz data is stored in two main structures:

- `Q` contains the question text, answer choices, correct-answer marker, explanation, and related lab settings.
- `M` maps common incorrect-answer categories to a misconception title and explanation.

The quiz contains six questions. `showQ()` renders the current question and answer buttons. When an answer is selected, the interface marks the correct answer, displays explanatory feedback, and tracks the user's first-try score and misconceptions.

The **Try it in the lab** button switches to the relevant simulation and applies the associated settings. The next-question button advances the quiz. At the end, the quiz shows the first-try score and any misconception categories triggered by incorrect answers, with an option to restart.

The score and quiz progress are held in JavaScript memory. They are not saved to a database or carried over to another browser session.

## 9. Navigation and animation loop

### `go(t)`

The `go()` function changes the active section and active navigation button by toggling the `on` class. It clears cached Canvas contexts so that canvases can be prepared again when needed, and scrolls to the top.

### Animation loop

The final `requestAnimationFrame()` loop repeatedly checks which section is active:

- Projectile: advance animation time when unpaused, then draw the trajectory and velocity graph.
- Orbit: draw and advance the orbit simulation.
- Forces: draw the ramp, forces, and acceleration graph.
- Quiz: quiz interactions are handled by their own event listeners.

Using `requestAnimationFrame()` lets the browser schedule drawing alongside screen refreshes. The code also caps the elapsed frame time used for projectile animation to reduce large jumps after a delayed frame.

When the window resizes, cached canvas contexts are cleared so they can be prepared again at the new dimensions.

## 10. Dependencies and deployment

The project is a static website. The application logic uses browser APIs and does not require a backend, package manager, or build tool.

The main external dependency is Google Fonts. If the font service is unavailable, the CSS specifies fallback fonts.

GitHub Pages can host the repository directly because `index.html` is in the repository root. To update the live site, commit and push the modified files to the configured publishing branch.

## 11. Testing ideas

Use these checks after changing the code:

1. Open each of the four navigation tabs and confirm the correct section appears.
2. Change each slider and verify the numerical values and drawings update.
3. In projectile mode, test zero degrees, a steep angle, a nonzero starting height, and nonzero air drag.
4. In orbit mode, compare launch speeds near 1.0 and 1.2 and switch integration methods.
5. In ramp mode, vary the angle and friction coefficient and check the transition from holding still to sliding.
6. Complete the quiz, verify feedback and scoring, use **Try it in the lab**, and restart the quiz.
7. Resize the browser and test the page on a narrow screen.
8. Check the browser console for errors after any code edits.

For more rigorous validation, add automated tests for the physics calculations and compare selected outputs against known analytical cases.

## 12. Current limitations and possible refactoring

- The entire application is contained in one HTML file, which is convenient for deployment but can become harder to maintain as it grows.
- The physics models use simplifying assumptions and numerical approximations; they are educational demonstrations rather than high-fidelity scientific tools.
- Canvas content is not inherently as accessible as equivalent HTML text. More detailed text descriptions and keyboard-accessible controls would improve accessibility.
- Quiz state is temporary and is not persisted.
- The orbit simulation uses normalized units, so its outputs are not directly real-world satellite predictions.
- Google Fonts requires an internet connection to load the preferred fonts, though fallback fonts are available.

A future version could move styles to `styles.css`, simulation code to separate JavaScript modules, and add automated tests without changing the core learning goals.
