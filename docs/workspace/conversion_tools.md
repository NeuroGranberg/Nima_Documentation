# Conversion Tools

## [dcm2niix](https://github.com/neurolabusc/dcm2niix)

 <kbd>**CLI**</kbd>

In my exploration of tools for BIDSification, I came across [**dcm2niix**](https://github.com/neurolabusc/dcm2niix), which is a crucial piece of software written in C++. It is designed to convert DICOM and PAR/REC files into the NIfTI format, a key step in preparing data for BIDS compliance. It's specifically engineered to handle brain imaging data conversions with minimal fuss, making it a favorite among researchers and clinicians alike. The software automatically generates JSON sidecar files for each NIfTI file it creates, which include important metadata necessary for BIDS datasets.

Among all, [**dcm2niix**](https://github.com/neurolabusc/dcm2niix) stands out for its efficiency in converting DICOM and PAR/REC files into NIfTI format. Its speed, ease of use, and broad compatibility make it a preferred choice when it comes to big data.

## [BIDS-converter](https://github.com/openneuropet/BIDS-converter)

 <kbd>**MATLAB and Python**</kbd>

BIDS-converter facilitates the conversion of PET images from DICOM and ECAT formats to NIfTI, leveraging both Matlab and Python scripts. It supports a wide range of PET imaging data, making it versatile for research and clinical use.

## [Heudiconv](https://heudiconv.readthedocs.io/en/latest/)

 <kbd>**Python**</kbd>

Heudiconv is tailored for converting DICOM files to the NIFTI format within a BIDS structure. Its tools are especially useful at the initial stages of BIDSification, facilitating the conversion of raw imaging data into a structured, standardized format.

## [nipype](https://github.com/nipy/nipype)

 <kbd>**Python**</kbd>

Nipype can use dcm2niix to convert images, showcasing the tool's utility in facilitating neuroimaging workflows across various programming environments.

## [dcmstack](https://github.com/moloney/dcmstack)

 <kbd>**Python**</kbd>

Dcmstack is a Python-based tool that converts DICOM images to NIfTI format while preserving metadata. It's particularly useful for researchers who need to retain study-specific information within the imaging files.

## [dicom2nifti](https://github.com/icometrix/dicom2nifti)

 <kbd>**Python**</kbd>

Dicom2nifti is a Python wrapper that utilizes the high-performance GDCMCONV executables for converting DICOM images to NIfTI. It's designed for efficiency and can handle a broad range of DICOM data.

## [dicm2nii](http://www.mathworks.com/matlabcentral/fileexchange/42997-dicom-to-nifti-converter)

 <kbd>**Matlab**</kbd>

This tool, developed in Matlab, allows for the scriptable conversion of DICOM images to NIfTI. Its Matlab foundation makes it highly adaptable for automated workflows and custom processing pipelines.

## [dicomtonifti](https://github.com/dgobbi/vtk-dicom/wiki/dicomtonifti)

 <kbd>**CLI**</kbd>

Dicomtonifti extends the capabilities of VTK, a widely used toolkit for 3D computer graphics, image processing, and visualization, to convert DICOM images to NIfTI format.

## [dinifti](http://as.nyu.edu/cbi/resources/Software/DINIfTI.html)

 <kbd>**CLI**</kbd>

Dinifti focuses on converting Siemens DICOM data to NIfTI, addressing the specific needs of researchers and clinicians working with Siemens imaging equipment.

## [DWIConvert](https://github.com/BRAINSia/BRAINSTools/tree/master/DWIConvert)

 <kbd>**CLI and GUI**</kbd>

DWIConvert specializes in converting DICOM images to NRRD and NIfTI formats, making it a valuable tool for researchers working with diffusion-weighted imaging data.

## [mri_convert](https://surfer.nmr.mgh.harvard.edu/pub/docs/html/mri_convert.help.xml.html)

 <kbd>**CLI**</kbd>

This tool is integrated within the FreeSurfer suite, known for its robustness in processing GE and Siemens data, though it may encounter difficulties with Philips 4D datasets.

## [nanconvert](https://github.com/spinicist/nanconvert)

 <kbd>**CLI**</kbd>

Nanconvert leverages the ITK library to convert DICOM images from GE and proprietary Bruker formats to standard formats like DICOM, addressing niche conversion needs.

## [Simple Dicom Reader 2 (Sdr2)](http://ogles.sourceforge.net/sdr2-doc/index.html)

 <kbd>**GUI**</kbd>

Simple Dicom Reader 2 utilizes dcmtk to read and convert DICOM images to the NIfTI format, providing a straightforward solution for DICOM processing.

## [spec2nii](https://github.com/wexeee/spec2nii)

 <kbd>**Python**</kbd>

Spec2nii is tailored for converting MR spectroscopy data to NIfTI, addressing the specific needs of spectroscopy analysis in research and clinical settings.

## [bidskit](https://github.com/jmtyszka/bidskit)

 <kbd>**Python**</kbd>

Bidskit utilizes dcm2niix to create BIDS datasets, streamlining the process with detailed guidance for users new to BIDS or neuroimaging data management.

## [BIDScoin](https://github.com/Donders-Institute/bidscoin)

 <kbd>**GUI, Python and CLI**</kbd>

BIDScoin offers a user-friendly GUI for converting DICOM images to BIDS format. Its comprehensive documentation supports users through the conversion process.

## [conversion](https://github.com/pnlbwh/conversion)

 <kbd>**Python**</kbd>

This Python library converts dcm2niix-created NIfTI files to the popular NRRD format, including DWI gradient tables. It highlights the versatility of dcm2niix in supporting various data formats.

## [DAC2BIDS](https://github.com/dangom/dac2bids)

 <kbd>**CLI**</kbd>

DAC2BIDS leverages dcm2niibatch to create BIDS datasets, providing an efficient path for researchers to organize their imaging data in line with BIDS standards.

## [dcm2bids](https://unfmontreal.github.io/Dcm2Bids)

 <kbd>**CLI**</kbd>

Dcm2Bids facilitates the creation of BIDS datasets using dcm2niix, supported by a comprehensive tutorial that guides users through the conversion process.

## [dcm2niir](https://github.com/muschellij2/dcm2niir)

 <kbd>**R**</kbd>

Dcm2niir is an R wrapper for dcm2niix/dcm2nii, extending the accessibility of DICOM to NIfTI conversion to the R programming community.

## [dcm2niixpy](https://github.com/Svdvoort/dcm2niixpy)

 <kbd>**Python**</kbd>

Dcm2niixpy is a Python package of dcm2niix, offering Python users direct access to dcm2niix's functionalities within their programming environment.

## [dcm2niix_afni](https://afni.nimh.nih.gov/pub/dist/doc/program_help/dcm2niix_afni.html)

 <kbd>**CLI, Python**</kbd>

This version of dcm2niix is included with the AFNI distribution, integrating DICOM to NIfTI conversion within a comprehensive suite of tools for brain imaging analysis.

## [dcm2niiXL](https://github.com/neurolabusc/dcm2niiXL)

 <kbd>**CLI**</kbd>

Dcm2niiXL is a shell script and optimized compilation of dcm2niix designed for fast conversion of extra-large datasets, addressing the needs of high-throughput imaging studies. dcm2niiXL is a script for running dcm2niix in parallel.

## [heudiconv](https://github.com/nipy/heudiconv)

 <kbd>**CLI, Python**</kbd>

Heudiconv can use dcm2niix to create BIDS datasets, facilitating the conversion of data acquired using the reproin convention into organized, shareable, and version-controlled datasets.

## [MRIcroGL](https://github.com/neurolabusc/MRIcroGL)

 <kbd>**GUI**</kbd>

Available for MacOS, Linux, and Windows, MRIcroGL provides a graphical interface for dcm2niix, facilitating the conversion of DICOM images in a user-friendly environment.

## [NeuroElf](http://neuroelf.net)

 <kbd>**GUI and MATLAB**</kbd>

NeuroElf can use dcm2niix to convert DICOM images, integrating this capability within its suite of tools for neuroimaging data analysis and visualization.

## [TractoR](http://www.tractor-mri.org.uk/TractoR-and-DICOM)

 <kbd>**R**</kbd>

TractoR (Tractography with R) uses dcm2niix for image conversion, integrating DICOM to NIfTI conversion within its suite of tools for advanced neuroimaging analyses and tractography with R.