---
layout: page
title: Software
permalink: /software/
---

Open-source tools for spinal cord MRI preprocessing, analysis, and quantification.

| Name | Purpose | Reference |
|---|---|---|
| **Processing**  |  |  |
|[Spinal Cord Toolbox](https://github.com/neuropoly/spinalcordtoolbox) | Segmentation of the spinal cord, gray matter, lesions; detection of anatomical highlights;  registration to templates; motion correction for diffusion and functional MRI time series; computation of quantitative MRI metrics, etc. | [De Leener et al. Neuroimage 2017](https://www.ncbi.nlm.nih.gov/pubmed/27720818) |
| [TotalSpineSeg](https://github.com/neuropoly/totalspineseg) | Tool for automatic instance segmentation of all vertebrae, intervertebral discs (IVDs), spinal cord, and spinal canal in MRI images | [Warszawer et al., 2025](https://www.researchgate.net/publication/389881289_TotalSpineSeg_Robust_Spine_Segmentation_with_Landmark-Based_Labeling_in_MRI) |
| [SpineReport](https://ivadomed.github.io/SpineReport/) | Automated extraction of spinal morphometrics and generation of structured radiological reports from MRI data. | [Molinier et al., 2026](http://arxiv.org/abs/2606.10021) |
| [T2*w SC MRI with navigator](https://github.com/NordicMRspine/MRINavigator.jl) | Navigator-based correction pipelines for demodulating time dependent field variations on spinal cord imaging. | [Beghini et al., Magn Reson Med. 2025](https://doi.org/10.1002/mrm.30475) |
| [Neptune - SC fMRI](https://github.com/rangadeshpande/Neptune) | User-interface-based MATLAB toolbox for processing spinal cord functional magnetic resonance imaging (fMRI) data. | [Rangaprakash & Barry, 2026](https://www.biorxiv.org/content/10.64898/2026.03.03.709443v1) |
| [Pastis - SC MR Spectroscopy](https://github.com/tngrssl/pastis) | Python package for the processing and quantification of single-voxel MR Spectroscopy data. Originally developed to reconstruct, process and quantify spinal cord MRS data at 7 T | [Roussel et al., Magn Reson Med. 2022](https://doi.org/10.1002/mrm.29182) |
| [7T-DSC-MRI-Toolbox - SC Perfusion / DSC](https://github.com/slevyrosetti/7T-DSC-MRI-Toolbox) | Python-based toolbox for the processing of cardiac-gated spinal cord Dynamic Susceptibility Contrast (DSC) perfusion MRI at 7T | [Lévy et al., Magn Reson Med. 2021](https://doi.org/10.1002/mrm.28559) |
| [MWI - not SC specific](https://jondeuce.github.io/DECAES.jl/dev/) | Julia tools for decomposing multi-exponential signals which arise from multi spin-echo MRI scans into exponential components | [(Doucette et al., Z Med Phys. 2020)](https://doi.org/10.1016/j.zemedi.2020.04.001) |
| **Templates** |  |  |
| [SC Template](https://github.com/neuropoly/template ) | Framework for creating unbiased MRI templates of the spinal cord |  |
| [PAM50](https://github.com/spinalcordtoolbox/PAM50 ) | T1w/T2w/T2*w template with 15WM/3GM bilateral parcels. Aligned with MNI ICBM152 template (Collins et al., 1999) | [De Leener et al., Neuroimage 2018](https://doi.org/10.1016/j.neuroimage.2017.10.041) |
| [AMU7T](https://github.com/spinalcordtoolbox/template-AMU7T) | T2*w/qT1 template with 15 WM + 8 GM bilateral and 4 inter-hemispheric parcels. | [Le Troter et al., 2023](https://archive.ismrm.org/2023/0569.html) |
| [PAMhisto](https://github.com/spinalcordtoolbox/PAM50/histology) | WM template from high resolution electron microscopy | [Duval et al., Neuroimage 2019](https://doi.org/10.1016/j.neuroimage.2018.10.033) |
| [Ex-vivo template](https://github.com/spinalcordtoolbox/exvivo-template ) | 9.4T T2 template from 13 ex-vivo specimens | [Gros et al., 2020](https://archive.ismrm.org/2020/1171.html) |
| **Synthetic SC MRI** |  |  |
| [SynSpine](https://www.nitrc.org/projects/synspine/ ) | DL model to generate SC synthetic data with varying atrophy levels | [Ganzetti et al., Front. Neuroinform. 2025](https://doi.org/10.3389/fninf.2025.1649440) |
| [LesionSCynth](https://github.com/rickymwalsh/LesionSCynth) | SC MS lesion synthesis method to improve DL segmentation model training | [Walsh et al., Imaging Neuroscience 2025](https://doi.org/10.1162/IMAG.a.1029) |


If your software is not listed, please fork [this repository](https://github.com/spinalcordmri/spinalcordmri.github.io) and submit a pull request.
