# Floating Sun-Tracking Solar Panel

An Arduino-based dual-axis solar tracking system designed for floating photovoltaic (FPV) applications, combining autonomous sun tracking with water-surface deployment capabilities.

## Overview

This project implements an intelligent solar tracking system optimized for floating solar panel installations. The system autonomously adjusts panel orientation to maximize solar energy capture throughout the day while maintaining stability on water surfaces.

### Key Features

- **Dual-axis sun tracking**: Azimuth and elevation control for optimal solar alignment
- **Floating platform compatible**: Designed for deployment on water bodies
- **LDR-based tracking**: Light-dependent resistor array for real-time sun position detection
- **Arduino control**: Microcontroller-based servo positioning system
- **Autonomous operation**: Self-adjusting without manual intervention
- **Energy efficiency**: Low-power operation suitable for off-grid deployment

## Motivation

Floating solar installations present unique challenges compared to ground-mounted systems. This project addresses:

- **Space constraints**: Utilizing water surface area for solar energy generation
- **Cooling benefits**: Water proximity improves panel efficiency through passive cooling
- **Sun tracking**: Compensating for panel movement due to water surface variations
- **Energy optimization**: Maximizing power output through continuous solar alignment

## System Architecture

```
┌─────────────────────────────────────────┐
│         Solar Panel (PV Module)         │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│      Dual-Axis Mounting System          │
│  ┌─────────────┐    ┌─────────────┐    │
│  │ Azimuth     │    │ Elevation   │    │
│  │ Servo Motor │    │ Servo Motor │    │
│  └──────┬──────┘    └──────┬──────┘    │
└─────────┼──────────────────┼───────────┘
          │                  │
     ┌────▼──────────────────▼────┐
     │    Arduino Controller      │
     │  ┌──────────────────────┐  │
     │  │  Tracking Algorithm  │  │
     │  └──────────────────────┘  │
     └────▲──────────────────▲────┘
          │                  │
     ┌────┴────┐        ┌────┴────┐
     │  LDR    │        │  LDR    │
     │ Array   │        │ Array   │
     │ (East/  │        │ (Top/   │
     │  West)  │        │ Bottom) │
     └─────────┘        └─────────┘
```

## Hardware Components

### Essential Components

| Component | Specification | Quantity | Purpose |
|-----------|--------------|----------|---------|
| Arduino Uno/Nano | ATmega328P | 1 | Main controller |
| Servo Motors | MG996R or similar (15+ kg-cm torque) | 2 | Azimuth and elevation control |
| LDR Sensors | GL5528 or equivalent | 4 | Light intensity detection |
| Resistors | 10kΩ | 4 | LDR voltage divider circuits |
| Solar Panel | 5-50W (application dependent) | 1 | Power generation |
| Floating Platform | Buoyant structure | 1 | Water deployment |
| Power Supply | 5V regulated or battery | 1 | System power |

## Technical Implementation

### LDR Sensor Configuration

The system uses four LDRs arranged in a cross pattern:

```
        Top LDR
           │
     ──────┼──────
     │     │     │
West LDR───┼───East LDR
     │     │     │
     ──────┼──────
           │
      Bottom LDR
```

### Tracking Algorithm

The control algorithm follows this logic:

1. **Read sensor values**: Sample all four LDR voltage readings
2. **Calculate differentials**:
   - Azimuth error = (East LDR - West LDR)
   - Elevation error = (Top LDR - Bottom LDR)
3. **Apply threshold**: Ignore small differences to prevent jitter
4. **Compute corrections**: Determine required servo adjustments
5. **Update positions**: Move servos to new angles
6. **Delay and repeat**: Wait for stabilization before next reading

### Servo Control

```
Azimuth Range: 0° to 180°
- 0°: East
- 90°: South (Northern Hemisphere)
- 180°: West

Elevation Range: 0° to 90°
- 0°: Horizontal
- 45°: Mid-day typical
- 90°: Vertical (not used)
```

## Software Details

### Pin Configuration

Default pin assignments (configurable in code):

```cpp
// Servo pins
const int AZIMUTH_SERVO_PIN = 9;
const int ELEVATION_SERVO_PIN = 10;

// LDR analog pins
const int LDR_EAST = A0;
const int LDR_WEST = A1;
const int LDR_TOP = A2;
const int LDR_BOTTOM = A3;
```

### Key Parameters

```cpp
// Sensitivity threshold (ignore differences below this value)
const int THRESHOLD = 50;  // ADC units (0-1023 scale)

// Maximum servo movement per iteration
const int MAX_STEP = 5;    // degrees

// Update interval
const int DELAY_TIME = 100; // milliseconds
```

## Installation

### Hardware Assembly

1. **Mount servos**: Attach azimuth servo to base, elevation servo to rotating platform
2. **Install solar panel**: Secure panel to elevation servo arm
3. **Position LDRs**: Mount sensors at panel edges with unobstructed sky view
4. **Wire connections**: Connect components according to circuit diagram
5. **Waterproof electronics**: Seal all connections for water deployment
6. **Prepare floating platform**: Ensure adequate buoyancy and stability

## Circuit Diagram

```
                    +5V
                     │
        ┌────────────┼────────────┐
        │            │            │
       10kΩ        10kΩ         10kΩ
        │            │            │
        ├─A0         ├─A1         ├─A2  ... (Arduino Analog Pins)
        │            │            │
       LDR          LDR          LDR
        │            │            │
        └────────────┴────────────┴──── GND

Servo Connections:
- Azimuth Servo:   Signal → D9,  Power → 5V,  GND → GND
- Elevation Servo: Signal → D10, Power → 5V,  GND → GND
```

## Operation

### Normal Operation

1. System powers on and initializes servos to default position
2. LDR sensors continuously monitor light intensity
3. Controller calculates error between opposing sensor pairs
4. Servos adjust panel orientation to minimize error
5. Panel achieves perpendicular alignment to sun
6. System repeats tracking cycle throughout daylight hours

### Night Mode

When ambient light falls below threshold:
- Servos return to morning start position (facing east)
- System enters low-power state
- Resumes tracking at dawn

### Water Deployment

Floating platform considerations:
- Ensure adequate waterproofing of all electronics
- Use marine-grade materials for exposed components
- Account for wave motion in mechanical design
- Consider anchor/mooring system to prevent drift
- Monitor panel stability and adjust counterweights

## Performance Optimization

### Expected Improvements

Compared to fixed-angle solar panels:
- **20-40% increase** in daily energy capture (latitude dependent)
- **Peak power tracking** maintained throughout day
- **Extended generation hours** from sunrise to sunset optimization

## Troubleshooting

### Common Issues

**Panel oscillates/jitters continuously**
- Increase `THRESHOLD` value in code
- Check for shadows falling on LDR sensors
- Verify stable power supply to servos

**Panel doesn't track sun**
- Verify LDR connections and functionality
- Test individual LDRs with multimeter
- Check servo operation with example sketches
- Ensure adequate sunlight for tracking

**Servos make noise but don't move**
- Insufficient power supply current
- Check servo load capacity vs. panel weight
- Verify servo connections and power rails

**System drifts on water**
- Implement anchor/mooring system
- Check platform balance and weight distribution
- Consider gyroscopic stabilization

**Corrosion or water damage**
- Improve waterproofing of enclosures
- Use conformal coating on PCBs
- Select marine-grade hardware
- Implement regular maintenance schedule

## Calibration Guide

### Initial Setup

1. **Mechanical alignment**:
   - Set azimuth servo to 90° (midpoint)
   - Set elevation servo to 45° (typical)
   - Manually align panel to south (Northern Hemisphere)

2. **LDR testing**:
   - Read all LDR values in Serial Monitor
   - Verify similar readings when equally illuminated
   - Replace any outlier sensors

3. **Threshold calibration**:
   - Start with `THRESHOLD = 50`
   - Observe tracking behavior
   - Increase if oscillation occurs
   - Decrease if tracking is sluggish

4. **Servo limits**:
   - Test full range of motion
   - Set software limits to prevent mechanical binding
   - Verify smooth operation throughout range

## Acknowledgments

This project combines solar tracking principles with floating photovoltaic technology to maximize renewable energy generation on water surfaces.

---

**Project Status**: Functional Prototype  
**Last Updated**: February 2026  
**Application**: Floating Solar PV Systems
