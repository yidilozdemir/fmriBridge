## fmriBridge
Creating a visual language of how neurological conditions change perceptions 

The aim of this project is to create a bridge of understanding of neurological conditions through visual simulations. I will 1) use openly shared neuroimaging datasets investigating perceptual differences in people with underlying mental health conditions, 
2) transform their fMRI data (process minimally, but in a scientifically sound way) to independent components of informative source signals, which of some will be shared across groups (ie "control" vs "with_depression").
3) Transform these shared and unshared components numerically as inputs to morph/recreate the initial object of fMRI study to simulate how these objects/concepts "look differently" for people with different cognitive mechanisms. 

The re-morphed objects, colors, audio will be explorative to show the differences between these populations, but scientifically uninformative for interpreting the object, due to limitations of how much we know of a correspondence between brain "activity" and underlying brain "representation". Yet this is not a concern for us, as it still allows us to translate what is mentally untranslatable; such as an emotional music experience of a person with depression, to something that we can all percieve in same terms: a morphed shape, a tweaked color, a happy birthday song in a different pitch! 

# data source 
openneuro.org: shared neuroimaging datasets, 
provisionally these data will come from two studies:
1) "Neural Processing of Emotional Musical and Nonmusical Stimuli in Depression" https://openneuro.org/datasets/ds000171/versions/00001 
Here, the data of interest is the brain activity associated with hearing a music in depressed individuals vs individuals never diagnosed with depression.
2) "A phenome-wide examination of neural and cognitive function" 
https://www.nature.com/articles/sdata2016110]
Here, the data of interest is the brain activity associated with percieving different colors in individuals diagnosed with schizophrenia vs ADHD vs bipolar disorder vs individuals never diagnosed with these conditions

I will use Datalad package to pull these data to my python environment, in which I will use scikit-learn and nipype to analyze my data

# 1) Data analysis pipeline, let's first understand the ML side

I will use a Tensor-probabilistic independent component analysis model to transform my data: 
This is a version of Indepent Component Analysis ( a type of unsupervised ML approach) that can reduce multidimensional; multi-subject, temporal and spatial (5D); data to common set of independent vectors in multiple axes of interest; temporally and spatially. 

Linear algebraically speaking, ICA reduces data to a set of orthonormal vectors, ie that are maximally independent in higher statistical dimensions, through mixing models of input that produce non-Gaussian probability distribution functions, and this non-Gaussianity is a marker for informative independent sources being mixed to create the observed output.

ICA in single-subject fMRI data can be used to decompose spatial and temporal aspects of signal to create independent components, 

fmri data: spatial * temporal (4D) data for a person ==> (spatial * Independent components) * (temporal * Independent components)

which is what neuroscientists need/use to understand brain function, for example by displaying the data through transforming it through the (temporal * ICA) matrix to see "when" an informative activity occurs, or by transforming it through (spatial * ICA) matrix to see "where" an informative activity occurs. 

However, my challenge here is to use this approach for multiple subjects, but still get the same independent vectors that reduce my data in a meaningful way; 

fmri data: spatial * temporal * subject (5D) data  ==>  ?? How do we get the same IC's that can decompose all 3 dimensions?

Concatenating two of the three higher dimensions (ex temporal* subject) then passing it through ICA can make sense for the given neuroscientific data, ex. resting state where the temporal aspect of the brain activity is not very informative between subjects anyways, but not here, so we need a smarter approach:

Tensor-probabilistic ICA:
Pass ICA on concatenating two higher dimensions, ex subject * temporal to create a maximally non-gaussian distribution of estimated spatial maps, then use singular value decomposition to reconstitute the mixing matrix (subject * temporal) to independent decomposed matrices and iterate the decompositions with pairs until the decomposed matrices match in different pairs of iterations

The resulting matrices will be my independent vectors that I can use to reduce my data

https://ieeexplore.ieee.org/abstract/document/1263605
https://www.fmrib.ox.ac.uk/datasets/techrep/tr04cb1/tr04cb1.pdf
https://www.fmrib.ox.ac.uk/datasets/techrep/tr04cb1/tr04cb1/node3.html


# 1) Data analysis pipeline, let's first understand the neuroside

To do this, I need to do some well-studied preprocessing steps to maximise the signal to noise ratio of my data and its fit across subjects

1) Mean-based intensity normalisation in individual subject's fMRI data: due to scanner physics; different locations in brain, based on proximity to channel coils in MRI which give the magnetic distortions, can be exposed to different strength of magnetic pulses which create skewed contrast values that need to be corrected

2)temporal filtering: Signal needs to be highpass-filtered to take out the signal with unusually low frequencies that can correspond to global scanner issues, or heat up etc

3)Spatial smoothing with gaussian weighted least squares minimisation: This will de-trend the voxels, aka physical 1mm^3 units of brain, which is needed due to nature of hemodynamic response&how it relates to neural activity

4) Mask for only relevant voxel activity; Need to "brain extract" the images to take out skull etc so that these signals do not introduce noise, and then need tissue segmentation to take out white matter and cerebrospinal fluid areas in brain where we do not expect information--not that we don't expect, it is just not possible due to nonexistence of neurons!

Then we can put these data through our ML machinery

# Results of data->neuroside+Ml-> spatial decomposition
I will use the independent vectors to transform my original data to a spatial representation projected through a matrix of subject * temporal, then apply stats here

# Stats for spatial decomposition
I will use a gaussian gamma model for voxel wise z scores. Normalgamma conjugate model reduces the influence of prior information in our posterior probabilities by reference priors and can be applied to model data with no known mean or variance

# 3) Use the z-stats as inputs to recreate my object

I will use use the z-stats as an array input in the dimensions of my simulated object. For example, for dataset 2, where the fMRI activity corresponds to a color, I will use this array input to make modifications to my RGB color codes to create colors for different populations; 1 for subject group of healthy patients, 1 for schizophrenia patients etc

For audio signals for dataset 1, I will use this array input as parameters for sampling rate, pitch etc.




