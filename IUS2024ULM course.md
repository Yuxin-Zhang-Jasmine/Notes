# SC-08 Super-resolution ultrasound imaging
### Pengfei Song and Olivier Couture, IEEE IUS 2024, Taipei, Taiwan
📺 [Video on Bilibili](https://www.bilibili.com/video/BV1L3DjYMEzs/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=e06c10e6def4a4103e4728dc5c00fbbb) ｜ 📑 [Slides](https://drive.google.com/file/d/1UKaK1_QazbT3JnrAsbBDLMSsKU4TnZUp/view?usp=sharing)

---

| Chapter                                       | Contents                                                                 |
|----------------------------------------------|--------------------------------------------------------------------------|
| Chapter 1: The sources of super-resolution imaging | 1. Imaging biological tissues  <br> 2. Ultrasound in biomedical imaging  <br> 3. Ultrasound contrast agents  <br> 4. Resolution in optics  <br> 5. Historic of super-resolution methods in ultrasound |
| Chapter 2: ULM, Its basic concepts | 1. Considerations on scanners  <br> 2. Microbubble administration  <br> **3. Localization of isolated microbubbles**  <br> **4. Tracking of flowing microbubbles**  <br> **5. Representation of data in ULM**  <br> **6. Extension to 3D Localization Microscopy** |
| Chapter 3: Applications of super-resolution imaging | 1. ULM in preclinical and clinical studies  <br> 2. Perspectives for ULM in biomedical imaging |
| Chapter 4: ULM, Advanced considerations | 1. Artificial Intelligence in ULM  <br> **2. Fundamental tradeoff in ULM**  <br> **3. Motion artifacts and correction methods**  <br> **4. Measures of spatial and temporal resolutions**  <br> **5. The PALA framework** |
| Chapter 5: Hands-on training                  | 1. Using PALA for your own ULM  <br> 2. Dataset and codes provided       |

---

## The Sources of Super-Resolution Imaging
### The landscape of microvascular imaging

<img src="https://github.com/user-attachments/assets/315337b3-4ca6-4a5f-9f08-9e5e9350f79a" alt="image" width="400">

### Ultrasound Localization Microscopy (ULM): a Super-Resolution UltraSound (SRUS) technique
  * Microbubbles are nearly fully surrounded by blood cells, flowing in vasculars
    * Red blood cells - 8 µm, $10^9 \text{ml}^{-1}$;
    * Microbubbles - 3 µm, $10^5 \text{ml}^{-1}$ 
  * High echogenicity is due to impedance mismatch and **resonance**
  * Performed with **near-resonant excitation** and **low acoustic pressure** to **maximizes oscillation** and  **minimize destruction** of the microbubbles
  * Microbubble oscillation is restricted in capillaries due to the confined space.
  * Nonlinear behaviour amplified by resonance → harmonic imaging
  
### Res/volution in optics --> ULM 
  * Nobel Prize in Chemistry 2014: super-resolved fluorescence microscopy, including PALM (PhotoActivated Localization Microscopy)
  * The speaker's prior technique (MUSLI, Couture et al. Patent 2010, IUS 2011 ) was directly inspired by PALM
  * In 2013, 3D approaches were introduced

<img src="https://github.com/user-attachments/assets/30604765-7f58-4e52-afbe-003f12c3c5e1" alt="image" width="1000">

---

## ULM Basic Concepts and Data Acquisition

### Fundamental tradeoff
* Low MB density → better localization
* High MB count → better image sampling
  
### Experiment advice
* Choose a **setup** that ensures **visible and numerous** microbubbles that can be **isolated**
* Administration routes:
  * Tail vein
  * Jugular vein catheter (more control; requires surgery)
  * Intraocular (easy; limited volume)
* Bolus injection
  * ✅ Easier technically (no pumps, more needle placement options, etc.)
  * ❌ Variable MB concentration (can degrade ULM reconstruction)
* Constant infusion
  * ✅ More consistent (benifit reconstruction)
  * ❌ Risk of MBs floating out of suspension
* General advice 
  * Less time under anesthesia = easier injection
  * Keep animal **warm**!
  * Let tail ‘hang’ for a bit to improve blood pressure
  * Tape everything down. Always.
    
### Clinical ULM imaging workflow
 * Limited to **2 bolus injections** of contrast agent
 * Prefer RF data saving over IQ (skip beamforming to get more frames)
 * Select an ROI to increase amount acquired
 * A quick “BubbleCheck” button:
   1. Acquire \~200 frames
   2. Beamform data
   3. SVD filter
   4. Display movie
 * Aberration correction is often necessary (for brain imaging)

<img src="https://github.com/user-attachments/assets/d88ce52e-be0b-4d6a-b3a9-707c2c5e37d3" alt="image" width="500">
<img src="https://github.com/user-attachments/assets/e7a589a3-abe2-4a84-b384-63513f873478" alt="image" width="500>

---
---

## ULM Data Processing

**Separation** of microbubbles from tissue

  * Nonlinear imaging (Harmonic?)
  * Motion filtering (High-pass or SVD)
  
**Localizing** sources (Deconvolution-like problem)

  * Interpolation-based
  * Gaussian fitting
  * Weighted average
  * **Radial symmetry extension**
    
**Tracking** MBs (an assignment problem) [Simpletracker and particle tracking toolbox](https://github.com/tinevez/simpletracker)
  
  * Why tracking :
    *  Filling gaps
    *  Remove duplicates
    *  Measure velocities & directions
     
**Alternative**: Track then localize (Leconte et al., 2023.)

**Motion Artifacts & Correction** methods:

  * non-rigid registration (in the kidneys: Hingot et al, 2017; Foiret et al, 2017)
    
    🔗 [Multimodality non rigid demon algorithm image registration](https://fr.mathworks.com/matlabcentral/fileexchange/21451-multimodality-non-rigid-demon-algorithm-image-registration) (Imregister, imregdemon, imregcorr, elastix, etc)
    
<img src="https://github.com/user-attachments/assets/14942959-4ee6-4625-8150-8d4d63a3ce4e" alt="image" width="330">
<img src="https://github.com/user-attachments/assets/caeb2b92-10e5-4c11-a3d2-8230518168b3" alt="image" width="330">
<img src="https://github.com/user-attachments/assets/b3ed57bb-c2fe-4b8c-b5b4-cc0882facb68" alt="image" width="330">

---

## Measures of Spatial and Temporal Resolutions

| Methods                                                                 | citation                               |
|--------------------------------------------------------------------------|----------------------------------------|
| The **Fourier Ring Correlation**                                         | Hingot et al, 2021; Heiles et al, 2022 |
| Using the stability along individual tracks                              | Hingot et al, 2021                     |
| Using simulations to estimate the localization precision                 | Heiles et al, 2021                     |
| Saturation curves to estimate temporal resolution                        | Huang et al, 2020                      |
| Using optical registration for confirmation                              | Huang et al, 2020                      |
| As the smallest separation between two features                          | Heiles et al, 2019                     |
| Using statistical separation between neighboring pixels                  | Errico et al, 2015                     |
| By comparing with the size of the tube                                   | Desailly et al, 2013                   |
| Using a theoretical estimation                                           | Desailly et al, 2013                   |

---

## From 2D to 3D (Volumetric ULM)

* Reduces user-dependency
* VolULM is mostly isotropic (like MRI/CT)
* Accurate quantification (velocity, direction, pulsatility)

<img src="https://github.com/user-attachments/assets/e4d9edc1-41ae-4047-8490-b76dc6570d54" alt="image" width="600">

**Optimizing volULM**

* Correct transducer position (neccessary on matrix transducers)
* Better acquisition sequences (spherical or/and cylindrical)
* Beamforming modifications

---

## Other perspectives

* **Going faster**
  * Higher MB concentrations (Huang et al., Scientific Reports 2020)
  * Increase SNR 
  * Separate interfering microbubbles
  * Separate tracks
  * Deep learning
    
* **Solving non-rigid motion**
  * Registrater MBs to a common tissue space
  * Compensate probe, breathing, and cardiac motion

* **Through the skull**
  * Use low frequencies + aberration correction
    
* **Beyond vessels**
  * What else structure to localize?
  * Vaporizing contrast agents / Nanovesicles / Nanobubbles
    
* **New super-resolution techniques**
  * rely on red blood cells for subwavelength imaging
    (Jensen, J.A. TUFFC 2024, You et al., P. Song, TUFFC 2023)
