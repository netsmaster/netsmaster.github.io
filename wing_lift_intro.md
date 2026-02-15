---
layout: default
hide_sidebar: true
---
# Wing Lift: Technical Design Notes

## Overview

Wing Lift is an interactive aerodynamics game where players draw airfoil shapes and receive real-time feedback on their aerodynamic performance. The game uses mathematical models from potential flow theory to analyze wing designs and compute performance metrics.

## Core Gameplay Flow

1. **Drawing Phase**: Players draw an airfoil shape on a canvas using mouse or touch input
2. **Fitting Phase**: The game automatically fits a Joukowski airfoil to the user's drawing
3. **Analysis Phase**: Aerodynamic properties are computed and visualized
4. **Scoring Phase**: A performance score is calculated based on lift, thickness, and volume

## Technical Architecture

### Mathematical Foundation

#### Joukowski Transformation
The game uses the **Joukowski transformation**, a conformal mapping from complex analysis that transforms circles into airfoil shapes:

```
z = ζ + 1/ζ
```

Where:
- `ζ` (zeta) represents a point in the circle plane
- `z` represents the corresponding point in the physical airfoil plane

This transformation allows the game to work with simple circle geometries while producing realistic airfoil shapes.

#### Potential Flow Theory
The aerodynamic analysis is based on **potential flow theory**, which models inviscid, incompressible flow around the airfoil. The flow is composed of:

- **Uniform flow**: Free-stream velocity
- **Doublet**: Models the airfoil's blockage effect
- **Vortex**: Implements the Kutta condition (smooth flow at trailing edge)

The circulation `Γ` is determined by the Kutta condition: `Γ = 4πU·y₀`, where `U` is the free-stream velocity and `y₀` is the circle center offset.

### Key Algorithms

#### 1. Shape Normalization (`extract.ts`)
- **PCA Alignment**: Uses Principal Component Analysis to align the airfoil chord horizontally
- **Resampling**: Uniformly resamples the drawn shape by arc-length
- **Envelope Extraction**: Bins the shape into x-stations to extract upper and lower surfaces
- **Thickness/Camber Calculation**: Computes maximum thickness and camber distributions

#### 2. Joukowski Fitting (`joukowski.ts`)
- **Two-Stage Search**: Coarse grid search followed by local refinement
- **Parameter Space**: Searches over `(x₀, y₀)` circle center offsets
- **Distance Metric**: Uses bidirectional Hausdorff distance to match user's drawing
- **Orientation Detection**: Automatically detects leading/trailing edges by analyzing vertex sharpness

#### 3. Streamline Computation (`streamlines.ts`)
- **RK4 Integration**: Uses 4th-order Runge-Kutta method to trace streamlines
- **Velocity Field**: Computes velocity at each point by inverting the Joukowski transformation
- **Boundary Handling**: Detects when streamlines approach the airfoil body or leave the domain

#### 4. Performance Scoring (`wing2.ts`)
- **Lift Coefficient (CL)**: Computed via pressure integration around the airfoil
- **Volume (Vol)**: Cross-sectional area of the airfoil
- **Thickness (Thick)**: Maximum vertical extent
- **Score Formula**: 
  ```
  score = -Thick·sin(Thick^1.5·4π/0.65) 
        + 10·(0.1·min(Vol,4.5)² - 0.83·Thick² + Thick^0.15) 
        + CL
  ```

### Implementation Details

#### Coordinate Systems
The game uses multiple coordinate systems:
- **Canvas Coordinates**: Raw pixel coordinates from user input
- **Physical Coordinates**: Joukowski-transformed coordinates for streamlines
- **Normalized Coordinates**: Chord-normalized coordinates (x ∈ [0,1]) for analysis

#### Validation
- **Self-Intersection Detection**: Ensures the drawn shape is valid
- **Aspect Ratio Check**: Prevents overly round shapes that can't be analyzed
- **Finite Point Validation**: Filters out invalid numerical results

#### Visualization
- **Canvas Rendering**: Uses HTML5 Canvas for real-time drawing
- **Transform Stack**: Applies multiple coordinate transformations for proper alignment
- **Streamline Visualization**: Renders flow field as red polylines after scoring

## Technical Stack

- **Frontend**: React + TypeScript + Canvas API
- **Build System**: Vite
- **Platform**: Reddit Devvit (runs in webview)
- **Mathematics**: Custom implementations of complex analysis algorithms

## Performance Considerations

- **Async Fitting**: Airfoil fitting runs asynchronously to avoid blocking the UI
- **Point Sampling**: Streamlines and distance calculations use adaptive sampling
- **Numerical Stability**: Careful handling of singularities near the airfoil body
- **Coordinate Transformations**: Efficient matrix operations for PCA and normalization

## Future Enhancements

Potential improvements could include:
- Real-time streamline visualization during drawing
- Multiple angle of attack analysis
- Drag coefficient calculations
- Comparison with standard airfoil profiles (NACA series)
- Export functionality for CAD software
