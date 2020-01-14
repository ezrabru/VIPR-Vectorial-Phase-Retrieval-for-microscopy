# PR-PixelWise

# Overview: 

This code is associated with the paper : “VIPR: Vectorial Implementation of Phase Retrieval for fast and accurate microscopic pixel-wise pupil estimation 2020”.

The code generates a phase mask from a set of images with pre-defined coordinates ( for a z-stack or any general design), optimized mainly for high NA objectives. 


# Note on future updates 

This demo is a current working version with 2 examples; in the near future a MATLAB GUI will be added for easier use, 
and later a JAVA GUI will be added. 

# Other software use:

•	The demo code uses bio-formats (bfmatlab) to load the demo images (the user needs to download it and add it to the path). 

•	Finobj.mat written by Yair M. Altman , used to handles the plots. 

# General guidelines: 

•	Evaluation and testing was done on a laptop with I7 8750H CPU and NVIDIA GeForce 2070RTX.

•	Coordinate system is defined like MATLAB images (x is right and y is down). 

# Work flow:

1)	Open ''Main''

2) Choose data set ( data_set = 1 for EPFL DH data and data_set = 2 for TP).

3)	Access the script ''VIPR_user_input'':
    This script contains all the required user data which is needed for ''Main.mat'' to run.

* Part A: this part contains the flags which control the MATLAB code outline:

    a)	Prior_mask_flag :(default 0) - 1 if you want to start from a pre-defined mask and not from a clear aperture. 

    b)	gpu_flag: 1(default) to run on GPU (CUDA) and 0 for CPU

    c)	vec_model_flag : 1(default) for vectorial model, 0 for scalar ( faster, use for air-objectives).

    d)	cost_function_flag : define cost function to optimize, 1-L1, 2-L2, 3- Poisson MLE, 4- Gaussian MLE (default-see paper) 

        if you want to add a new cost, consult with the author, it is quite easy to do. 

    e)	plot_flag: 1 to visualize results during optimization(default), slows down the code.
    
    f)	Alg_flag: choose gradient descent scheme,  1 - ADAM (default), 2 - Adamax , 3- Nadam, 4 - Nesterov, 5- Vanilla SGD

    g)	vec_model_pol : needed If vec_model_flag=1, choose polarization state ‘x’(default), ‘y’, or ‘b’ for both. 

    h)	noisy_flag: 0- for design PSFs, 1 - for real measurements(default).

* Part B: define the optical system and measured PSFs. 

    Open ''init_input.mat'' and change the optical parameters to match your setup. 

    Notes: for freely rotating dipole, leave polarization vector as zeros. 

* Part C: more advanced optimization parameters. 

    a)	IS.I_thr_flag : how to  threshold the data - 1 is for thresholding pixels below IS.thr*max(I) per image I, 2 - threshold pixels         below IS.thr*background_std.

    b)	IS.I_thr: threshold parameter

    c)	IS.corner_size : size of corners to  estimate noise [pixels]

    d)	IS.SGDiter : how  many iterations to SGD (default 300)

    e)	IS.step_size : step  size (try 3e-1 for ADAM and 3e-8 for SGD)

    f)	IS.point_num : size of mini-batch per SGD iteration (default 3)

    g)	est_gBlur_flag : 1(default)- estimate gBlur after 1/3 of the iterations, 0 - leave initial guess

    h)	IS.gBlur_cost : cost function  to estimate the gaussian blur kernel if est_gBlur_flag=1, (1-4 same as cost_function_flag , 5 -           by corr2)

    i)	IS.last_iter : how many iterations  to run not with SGD (at end of optimization), at these iterations, the noise and blur are           not randomized. 

    j)	IS.last_iter_flag : 1 - contuine SGD, 2 - global gradient, 3- batch the lowest correlation points, 4- adaptive sampling with             side info on corr (Gopal, Siddharth. "Adaptive sampling for SGD by exploiting side information." International Conference on             Machine Learning. 2016)

    k)	IS.thr_corr : threshold for correlation calc (used if last_iter_flag = 3 or 4)

    l)	IS.upsample_fact : (default 1) if you wish to upsample the data, usufull if object space pixels are large compared to the               wavelength

    m)	IS.update_Signal : 1 - update signal at second half of iterations (needs more iterations, but is more accurate - might overfit           the data), 0 - keep the image sum as initial guess

    n)	IS.plotsize : size of psf plots [pixels]

* Part D: load the data set

    • For your code -  change this function to your own code such that the variable IMG will contain the 3d matrix of the z-                   stack(recommended to use an odd grid size ) 

      and that the variable z_stack_pos will contain the vector of NFP positions for the reconstruction. 

      Default: for data_set = 2 :the code opens the folder ‘’TP images’’ and reads the Tiff images starting with the letter ‘’T’’.

      ![](gifs/TP_PSF.gif)

      We added the z position of the images to the file metadata, to insert your data, remove this line and load the positions in any         other way. 

      for data_set = 1 :the code opens the folder ‘’EPFL DH data’’ and reads images using the activations csv.
        
      ![](gifs/DH_PSF.gif)

* Part E: optional coordinate entry.

    Default: only a z-stack was measured. But the code can handle any input of x,y,z,NFP coordinates.


# Output (of the script ''Main'')

Plots (if plot_flag = 1)

a)	A phase mask plot will be seen (with modulus of 2*pi), plotted every 30 iterations

b)	Sample PSFs are plotted with the matching model, plotted every 30 iterations

c)	The cost function value, plotted every 30 iterations

d)	Correlation between the model and the measured stack, plotted at the end

e)	A slow visual comparison between the measured stack and model, plotted at the end


Variables output:

a)	maskRec – the phase retrieved mask, unwrapped. 

b)	gB – std of thegaussian blur kernel estimation [pixels]

c)	Nph – vector of estimated image intensities 

d)	I_mod – the reconstructed z-stack.

# Use after retrieval

The function PSF_generator can be used estimate the recovery by creating an in-line function at the end of the script:

PSF_Gen = @(phase_mask,x,y,z,NFP) PSF_generator(phase_mask,[x,y,z],IS,NFP,opt_phase_mat...
    ,g_bfp,circ,circ_sc,int_cos,vec_model_flag);
 
note: the generator outputs an image of sum 1 ( before cropping ), no background and unblurred.

# Citation

If you use this code for your research, please cite our paper:
```
@article{ferdman2020vipr,
  title={VIPR: Vectorial Implementation of Phase Retrieval for fast and accurate microscopic pixel-wise pupil estimation},
  author={Ferdman, Boris and Nehme, Elias and Weiss, Lucien E and Orange, Reut and Alalouf, Onit and Shechtman, Yoav},
  journal={bioRxiv},
  year={2020},
  publisher={Cold Spring Harbor Laboratory}
}
```

# Contact

To report any bugs, suggest improvements, or ask questions, please contact me at "borisferd@gmail.com"
 





