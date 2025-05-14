# SC-08 Super-resolution ultrasound imaging
### Pengfei Song and Olivier Couture, IEEE IUS 2024, Taipei, Taiwan
[Video on Bilibili](https://www.bilibili.com/video/BV1L3DjYMEzs/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=e06c10e6def4a4103e4728dc5c00fbbb) | [Slides](https://drive.google.com/file/d/1UKaK1_QazbT3JnrAsbBDLMSsKU4TnZUp/view?usp=sharing)
| Chapter                                       | Contents                                                                 |
|----------------------------------------------|--------------------------------------------------------------------------|
| Chapter 1: The sources of super-resolution imaging | 1. Imaging biological tissues  <br> 2. Ultrasound in biomedical imaging  <br> 3. Ultrasound contrast agents  <br> 4. Resolution in optics  <br> 5. Historic of super-resolution methods in ultrasound |
| Chapter 2: ULM, Its basic concepts | 1. Considerations on scanners  <br> 2. Microbubble administration  <br> **3. Localization of isolated microbubbles**  <br> **4. Tracking of flowing microbubbles**  <br> **5. Representation of data in ULM**  <br> **6. Extension to 3D Localization Microscopy** |
| Chapter 3: Applications of super-resolution imaging | 1. ULM in preclinical and clinical studies  <br> 2. Perspectives for ULM in biomedical imaging |
| Chapter 4: ULM, Advanced considerations | 1. Artificial Intelligence in ULM  <br> **2. Fundamental tradeoff in ULM**  <br> **3. Motion artifacts and correction methods**  <br> **4. Measures of spatial and temporal resolutions**  <br> **5. The PALA framework** |
| Chapter 5: Hands-on training                  | 1. Using PALA for your own ULM  <br> 2. Dataset and codes provided       |

## Chapter 1: The sources of super-resolution imaging
### The landscape of microvascular imaging

![image](https://github.com/user-attachments/assets/315337b3-4ca6-4a5f-9f08-9e5e9350f79a)

### Ultrasound Localization Microscopy (ULM): a Super-Resolution UltraSound (SRUS) technique
  * Microbubbles are nearly fully surrounded by blood cells, flowing in vasculars
    * Red blood cells - 8 µm, $10^9 ml^{-1}$;
    * Microbubbles - 3 µm, $10^5 ml^{-1}$ 
  * High echogenicity is due to impedance mismatch and **resonance**
  * Performed with **near-resonant excitation** and **low acoustic pressure** to **maximizes oscillation** and  **minimize destruction** of the microbubbles
  * Microbubble oscillation is restricted in capillaries due to the confined space.
  * The nonlinear behaviour is also amplified due to resonance --> harmonic imaging
  
### Res/volution in optics --> ULM 
  * The Nobel Prize in Chemistry 2014, super-resolved fluorescence microscopy, including PALM (PhotoActivated Localization Microscopy)
  * The speaker's prior technique (MUSLI, Couture et al. Patent 2010, IUS 2011 ) was directly inspired by PALM
  * In 2013, 3D approaches were introduced
  
  ![image](https://github.com/user-attachments/assets/30604765-7f58-4e52-afbe-003f12c3c5e1)
  
## Chapter 2: ULM basic concepts
### Experiment advice
* Choose the **setup** as long as you end up with **visible and numerous** microbubbles that can be **isolated** from each others
* tail vein injections / Jugular vein catheter (more control; requires surgery) / Intraocular (relatively easy; limited volume)
  1. Bolus injection:
    * pros: Technically easier (no pumps, more needle placement options, etc.)
    * cons: Variable MB concentration can degrade ULM reconstruction
  2. Constant infusion:
    * pros: More consistent ULM reconstruction
    * cons: Additional concerns such as MBs floating out of suspension
* General advice 
  * Less time under anesthesia = easier injection
  * Keep the animal **warm**!
  * Letting the tail ‘hang’ for a bit can improve blood pressure
  * Tape everything down. Always.
    
### Clinical ULM imaging workflow (in Chapter 3)
 * We were limited to **2 bolus injections** of contrast agent
 * RF data saving is preferred over IQ (skip beamforming to get more frames)
 * Select an ROI to increase amount acquired
 * We put together a quick “BubbleCheck” button:
   * Acquire ~200 frames
   * Beamform data
   * SVD filter
   * Display movie
 * Aberration correction is often necessary (for imaging the brain)
   
![image](https://github.com/user-attachments/assets/d88ce52e-be0b-4d6a-b3a9-707c2c5e37d3)
![image](https://github.com/user-attachments/assets/e7a589a3-abe2-4a84-b384-63513f873478)



### Data processing
* **Separation** of microbubbles from tissue
  * Nonlinear imaging (harmonic?)
  * Motion (High-pass or SVD Filtering)
* **Localizing** a source (a form of deconvolution)
  * Interpolation based methods
  * Gaussian fitting method
  * Weighted average based methods
  * **Extension to radial symmetry**
* **Tracking** MB (an assignment problem) [Simpletracker and particle tracking toolbox](https://github.com/tinevez/simpletracker)
  * Why tracking for ULM
    *  filling the gaps
    *  remove duplicates
    *  measure velocities and directions
* An alternative approach: tracking then localization (Leconte et al., 2023.)

### From 2D to 3D (Volumetric ULM)
* 3D can reduce user-dependency
* As in MRI or CT, volULM is mostly isotropic
* In volULM, quantification (velocity, direction) is accurate
* In volULM, Pulsatility can be assessed

![image](https://github.com/user-attachments/assets/e4d9edc1-41ae-4047-8490-b76dc6570d54)

* How to obtain optimal volULM:
  * Correcting for the transducer position is necessary on matrix transducers
  * Improvements in acquisition (Spherical or/and Cylindrical sequence)
  * Modifying Beamforming

### Other perspectives
* Going faster
  * Higher concentrations (Huang et al., Scientific Reports 2020)
  * Increase SNR 
  * Separate interfering microbubbles
  * Separate tracks
  * Deep learning
* Solving non-rigid motion
  * Registration of microbubbles on a common tissue space
  * Probe motion
  * Breathing 
  * Cardiac motion
* Penetrating the skull easily (Low frequencies, Aberration correction, ..)
* Leaving vessels (What structure to localize? Vaporizing contrast agents / Nanovesicles / Nanobubbles)
* New super-resolution techniques rely on red blood cells for subwavelength imaging (Jensen, J.A. TUFFC 2024, You et al., P. Song, TUFFC 2023)
