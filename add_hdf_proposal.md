GSoC'25 Proposal - TARDIS RT Collaboration

<!-- <div style="display: flex"> -->
![gsoc banner](./GSoC-logo-horizontal.svg) 
<!-- </div> -->

# Adding HDF Writing Capabilities to TARDIS Modules

## Project Details

- Organization : [TARDIS RT Collaboration](https://github.com/tardis-sn)
- Project Title : [Adding HDF Writing Capabilities to TARDIS Modules](https://tardis-sn.github.io/summer_of_code/ideas/#adding-hdf-writing-capabilities-to-tardis-modules)

- Mentors : [Andrew Fullard](https://github.com/andrewfullard), [Atharva Arya](https://github.com/atharva-2001), [Abhinav Ohri](https://github.com/KasukabeDefenceForce)

- Difficulty : HARD

## Personal Information

Name : Karthik Rishinarada

Email : karthikrk11135@gmail.com

LinkedIn: [Karthik Rishinarada](https://www.linkedin.com/in/karthik-rishinarada-a61b39251/)

Github id : [karthik11135](https://github.com/karthik11135)

Brief Summary : I'm a senior-year college student from NIT Trichy, India. I interned at Capital One, where I worked with a Python team. Over the past few months, I have developed an interest in open source and started exploring open-source codebases. I enjoy listening to music, window shopping, and planning vacations.

Resume : [Link to my Resume](https://drive.google.com/file/d/1LTs82Yv-aLM0iVrQHyoPOCQsfxLe_wFC/view?usp=sharing)

## <u>About the Organisation</u>

TARDIS (Temperature And Radiative Diffusion In Supernovae) is an open-source radiative transfer code that is developed for simulating supernova spectra. TARDIS helps astrophysicsts understand how exploding stars appear to observers by creating synthetic spectra - essentially predicting what these star explosions would look like through a telescope. Rather than taking days or weeks to simulate a spectrum, TARDIS uses real approximations to generate while maintaining accuracy.

<u>Core Components of TARDIS</u>
| **Components** | **Description** |
|-------------------------------------|------------------------------------------------------------------|
| **Configuration & Input** | <ul><li>Handles model parameters, atomic data, and abundance configurations (YAML or HDF).</li></ul> |
| **Physical Model Setup** | <ul><li>Establishes the radial grid structure (ex: Radial 1D Geometry) and initial conditions.</li><li>Contains density profiles and abundance distributions across shells.</li></ul> |
| **Monte Carlo Radiation Transport** | <ul><li>Implements core Monte Carlo setup for packet propagation through shells.</li><li>Handles packet interactions including scattering, absorption, and re-emission.</li></ul> |
| **Plasma Physics** | <ul><li>Calculates ionization states and level populations.</li><li>Handles atomic transitions and electron density calculations.</li></ul> |
| **Opacity Calculation** | <ul><li>Computes line and continuum opacities for each shell.</li>
| **Monte Carlo Iteration Handler** | <ul><li>Controls the main iteration loop for temperature convergence. Once the convergence happens the data is used to update plasma properties</li></ul> |
| **Spectrum Synthesis** | <ul><li>Collects energy packets into observable spectra.</li>

<u> Flow Diagram </u>
![flow diagram of tardis](./tardisflow.png)

## Project Summary

The current implementation of HDFWriterMixin class is very limited. In the HDF file it can only store data of the following :
1. Scalars are stored in the Series under the path ```/scalars```.
2. Basic unit conversions to CGS.
3. 1D arrays are stored as ```pd.Series```.
4. 2D arrays are stored as dataframes. 
   
Reference : [HDFWriterMixin](https://github.com/tardis-sn/tardis/blob/6d63ee88f81a39611f0a6f8a84e07d8442bb64dd/tardis/io/util.py#L191)

The goal of this project is to enhance the capabilities of the HDF writing. It aims to build simulation state preservation, restoration and incorporate checkpoint integration with regression testing. 

Benefits from the project
1. Simulation state snapshot is stored accurately from future references. In this way, they can be shared with others. 
2. Two simulation states can be compared
3. State reconstruction from an existing HDF file enables users to use the instance directly. 
4. Automatic checkpoint integration enables to create HDF files whenever the condition occurs. 
5. Debugging capabilities are improved
6. The regression testing techqniues are most robust with the checkpoints.

| **TARDIS Module**       | **State Storage**                                                                 | **Use cases** |
|-------------------------|----------------------------------------------------------------------------------|-------------|
| **Model Component**     | Geometric configurations, Abundance distributions, Density profiles, Temperature information | Enables simulation state restoration; Allows comparison of different model configurations. |
| **Plasma Module**       | Ion and Level populations, Plasma properties (DilutePlanckianRadiationField, HeliumTreatment, JBlues, etc.) | Preserves complex plasma states for analysis; Allows comparison across iterations. |
| **Transport Component** | Interaction of photons, Energies, Opacity states, etc.                          | Monte Carlo iterations can be restarted; Helps analyze packet propagation patterns. |
| **Opacity Component**   | Line opacity data, Continuum opacity data                                       | Facilitates analysis of opacity updates during iterations. |
| **Spectrum Component**  | Line formation regions, Spectra at different iterations                         | Supports spectral evolution comparison; Helps analyze line formation processes. |



---

## First Objective Task
Task : Add a method to the spectrum class that allows restoring the class from an HDF. Share the notebook in a pull request.

Link to solution : [GSOC PR](https://github.com/tardis-sn/tardis/pull/2995)

Explanation : from_hdf method is created on the TARDISSpectrum class to retrieve all the data from the hdf file and store as an object. The two main options that are required to initialize the spectrum class are the frequencies and luminosities. In the HDF file they are stored in the paths ```'/tardis_spectrum/_frequency'``` and ```'/tardis_spectrum/luminosity'``` respectively. The data reciding in these paths is retrieved and the class is created.  


---

## My Approach towards the project

A flexible schema can be created to store all data in HDF files. A base serialization class can be implemented to serialize complex data structures. Module-specific serialization can be implemented for all modules (e.g., Plasma, Transport, etc.).

For checkpoints, a checkpoint manager can be implemented, utilizing various trigger algorithms (e.g., iteration based, time based, storage based, convergence based, etc.). Whenever a trigger condition is occured, a checkpoint is created. Each checkpoint is stored as an HDF5 file containing the entire simulation state. Iteration specific data can be retrieved from these files. Finally, necessary unit tests can be written, and proper documentation of code changes are to be updated.

```
/checkpoints/
    simulation_checkpoint_iter_100.h5
    simulation_checkpoint_iter_200.h5
    simulation_checkpoint_iter_300.h5
```

Example of how a checkpoint file structure could be : 
```
checkpoint.h5
├── metadata/
│   ├── timestamp
│   └── iteration_number
├── simulation_state/
│   ├── plasma_state
│   ├── transport_state
│   └── atomic_data_references
└── convergence_data/
    ├── t_rad
    ├── w
    └── electron_densities

```

Here is a code example for a trigger: 
```
class ConvergenceTrigger:
    def __init__(self):
        self.last_state = None
    def checkpoint_trigger(self, t_rad, w, el_densities):
        current_state = {
            't_rad': t_rad,
            'w': w,
            'electron_densities': electron_densities
        }
        if self.convergence_changed(current_state):
            return True
        return False

# In this example, the checkpoint trigger returns True when convergence occurs storing the Transport State, Plasma State, and other relevant data.
```

To restore a simulation instance from an HDF5 file, module-specific logic can be implemented. The restoration process involves:

1. Loading the checkpoint (HDF5) file.

2. Retrieving the component states.

3. Reconstructing objects using the stored data.

4. Resuming the simulation from the saved state.

```
def from_checkpoint(hdf_file):
    with h5py.File(hdf_file, r) as f:
        transport_state = f['transport_state']
        return TransportState.restore_class(transport_state)

def restore_class(cls, state):
    # Retrieve all the transport data from the file
    opacity_state = state['opacity_data']
    packet_data = state['packet_data']
    return cls(opacity_state, packet_data)
```

![approach diagram](./hdfapproachdiagram.png)

---

## Milestones
Here's a detailed breakdown of milestones, each milestone's description, and deliverables:

| Milestones          | Description                               | Deliverables |
|--------------------|-------------------------------------------|--------------|
| Trigger Framework | Creating trigger architecture. Implementing different types of triggers | - `CheckpointTrigger` base class  </br> - Trigger  classes for iteration based, time based and convergence based |
| Restoration Framework  | Create base restoration class. Implement module specific logic   | - Functional restoring logic </br> - Restoring funcitons for plasma, transport and simulation classes </br> - Unit tests written |
| Testing and Documentation      | Unit and Integration tests   | - Unit tests for all the code written  </br> - Integration tests </br> - Proper documentation for the updated code|

---

### Why I've chosen TARDIS?

Over the past few months I wanted to explore open-source contributions. So I started looking for python repositories on GitHub. That’s when I came across TARDIS.

I've gone through the entire codebase and gained an understanding of its main components. While going through the code, I encountered numerous scientific terms like Monte Carlo Iteration, Plasma State, JBlues etc. I really enjoyed the process of googling these terms and understanding them. The documentation was written in a very clear way. The clarity of explanations in the documentation made it easier to understand the code. These are the two primary things that made me stick to contributing to TARDIS.

---

### Why am I the best fit?

My first internship offer was from a fintech company — Capital One, where I worked as a Software Developer Intern for two months. During this time, I improved my Python programming skills, which is the primary language used in TARDIS. I improved the runtime of our codebase by 40%. After my internship, I discovered TARDIS and began exploring its code by studying the documentation and working through the quickstart tutorials. I've gone threw few issues in the codebase and tried to solve a few of them.

Main Tardis repo:

- [add test to line info](https://github.com/tardis-sn/tardis/pull/2947)
- [from Simulation test for RPacket Plot](https://github.com/tardis-sn/tardis/pull/2945)
- [TODO: Tests of Composition](https://github.com/tardis-sn/tardis/pull/2944)
- [TODO: Rename tau to tau_event](https://github.com/tardis-sn/tardis/pull/2931)
- [Tests for config validator](https://github.com/tardis-sn/tardis/pull/2926)
- [refactor: Remove ConfigWriterMixin class](https://github.com/tardis-sn/tardis/pull/2926)

Regression Data repo:

- [Regression data for Composition](https://github.com/tardis-sn/tardis-regression-data/pull/44)

Carsus repo:

- [fixes NISTWeightsComp initialization error](https://github.com/tardis-sn/carsus/pull/436)

Apart from this, I actively explore new technologies and work on personal projects to enhance my skills. I believe that I am a perfect fit for this project because I've a good grip on the codebase and I understand what happens under the hood. I also believe that I can hit the ground running from day 0.

---

### Conclusion

In conclusion, my strong knowledge in Python, experience with open-source contributions, and familiarity with the TARDIS codebase make me a great fit for this project. While I have exams from May 3rd to May 9th, I will be entirely free afterward and fully committed to executing the project. I look forward to contribute meaningfully to the TARDIS RT Collaboration.
