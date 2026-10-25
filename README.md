# STEMLab

**Interactive physics experiments for learning mechanics through exploration.**

[**Open the live website →**](https://asmaareesha.github.io/StemLab/)

STEMLab is a browser-based learning project by **Asma Areesha Hanifi**. It lets learners change physical parameters, observe simulations, inspect graphs, and test their understanding of mechanics.

## What it includes

- **Projectile motion:** Explore how launch speed, angle, starting height, gravity, and air-drag settings affect a projectile's path.
- **Satellite orbits:** Change launch speed and compare Leapfrog and Euler numerical integration methods. Observe the trajectory and relative energy error.
- **Forces on a ramp:** Adjust the ramp angle, mass, and friction coefficient to inspect weight, normal force, friction, net force, and acceleration.
- **Knowledge-check quiz:** Answer conceptual questions, receive explanations, review misconceptions, and jump from a question into a related simulation.
- **Interactive graphs:** Scrub through projectile motion and change the ramp angle from its graph.
- **Responsive interface:** The layout adapts to smaller screens, with a light/dark color scheme that can respond to the device's color preference.

## Built with

- HTML for the page structure
- CSS for styling, layout, responsive behavior, and theme colors
- Vanilla JavaScript for simulation logic, controls, quiz behavior, and animation
- HTML Canvas 2D for drawing trajectories, graphs, force arrows, and the orbit
- Google Fonts for Atkinson Hyperlegible and Bricolage Grotesque

The project does not use a JavaScript framework or require a package installation/build step. Google Fonts are loaded from an external service; system fonts provide fallbacks.

## Try it

1. Open the [live STEMLab website](https://asmaareesha.github.io/StemLab/).
2. Choose **Projectile**, **Orbit**, **Forces**, or **Check your thinking**.
3. Change the sliders and observe how the visualization and numerical readouts respond.
4. Use the “Try this” prompts to make a prediction before changing the values.
5. In the quiz, read the explanation and select **Try it in the lab** to explore a related concept.

### Suggested experiments

- **Projectile:** Compare launch angles of 30° and 60° with the same speed and no air drag. Why can the ranges match in the idealized model?
- **Orbit:** Compare a launch speed of 1.0 with 1.2. How does the path change?
- **Numerical methods:** Switch between Leapfrog and Euler and compare the energy-error graph.
- **Forces:** Increase the ramp angle and identify when the block begins to slide for a given friction coefficient.

## Run it locally

STEMLab is a static website, so no dependencies need to be installed.

1. Download or clone this repository.
2. Open `index.html` in a modern browser.

To clone it with Git:

```bash
git clone https://github.com/Asmaareesha/StemLab.git
cd StemLab
```

Then open `index.html` in your browser.

For a more convenient development workflow, use a local editor with a static-file preview or local server. The website itself does not require a server-side application.

## Project structure

```text
StemLab/
├── index.html              # HTML, CSS, JavaScript, simulations, and quiz
├── README.md               # Project overview and getting-started guide
└── CODE_WALKTHROUGH.md     # Detailed explanation of the implementation
```

The current implementation keeps the application in one HTML file. This makes it straightforward to host as a static site, although splitting the CSS and JavaScript into separate files could make future maintenance easier.

## How it works

The page is organized into four sections controlled by navigation buttons. JavaScript calculates the physical state, and Canvas draws the result. Sliders update simulation parameters and trigger recalculation. A continuous animation loop redraws the active experiment.

Read [`CODE_WALKTHROUGH.md`](CODE_WALKTHROUGH.md) for a detailed explanation of the structure, state objects, simulation methods, drawing functions, quiz flow, and limitations.

## Accuracy and limitations

STEMLab is an educational model, not a professional physics simulator. It uses simplified assumptions and normalized units in the orbit simulation. Numerical integration introduces approximation error, and the simulated results depend on the assumptions and step sizes in the code. The air-drag model is simplified.

Use the experiments to build intuition and investigate relationships, and compare the results with equations or course materials when precision matters.

## Accessibility and usability

The interface includes labels for canvas regions, visible keyboard-focus styling for buttons and inputs, readable type choices, and a responsive layout. The motion behavior also checks the device's reduced-motion preference. Canvas drawings themselves are visual outputs, so equivalent text-based descriptions could be expanded in a future version.

## Future improvements

Possible next steps include:

- Add a dedicated explanation panel with the equations used in each simulation.
- Add automated checks for key calculations and edge cases.
- Separate HTML, CSS, and JavaScript into maintainable files.
- Improve keyboard and screen-reader access to information currently shown only in Canvas.
- Add an explicit pause/restart control and more complete motion controls to each experiment.
- Document model assumptions and compare numerical results with analytical solutions.

## Author

**Asma Areesha Hanifi**

- Live project: https://asmaareesha.github.io/StemLab/
- Source code: https://github.com/Asmaareesha/StemLab

## License

No license is currently specified in this README. Unless a license is added to the repository, visitors should not assume that they have permission to redistribute or modify the code. If you want others to reuse the project, choose a license and add a `LICENSE` file.
