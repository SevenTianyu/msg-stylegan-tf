# 文献【42】代码
# MSG-STYLEGAN-TF
## Official code repository for the paper "MSG-GAN: Multi-Scale Gradients for Generative Adversarial Networks" [[arXiv]](https://arxiv.org/abs/1903.06048)
<p align="center">
<img alt="Teaser Diagram" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/teaser.jpg" />
<br>
</p>

### Why this repository?
Our previous research work released the 
[BMSG-GAN](https://github.com/akanimax/BMSG-GAN) code in PyTorch 
which applied our proposed multi-scale connections in the basic
ProGAN architecture (i.e. DCGAN architecture) 
instead of using the progressive growing.
This repository applies the Multi-scale Gradient connections 
in StyleGAN replacing the progressive growing used 
for training original StyleGAN. The switch to Tensorflow was 
primarily to ensure an apples-to-apples comparison with StyleGAN. 

### Due Credit
This code heavily uses NVIDIA's original 
[StyleGAN](https://github.com/NVlabs/stylegan) code. We accredit and
acknowledge their work here. The 
[Original License](https://github.com/akanimax/msg-stylegan-tf/blob/master/LICENSE_ORIGINAL.txt) 
is located in the base directory (file named `LICENSE_ORIGINAL.txt`).

### Abstract
While Generative Adversarial Networks (GANs) have seen huge 
successes in image synthesis tasks, they are notoriously 
difficult to adapt to different datasets, in part due 
to instability during training and sensitivity to hyperparameters. 
One commonly accepted reason for this instability is
that gradients passing from the discriminator to the 
generator become uninformative when there isn’t enough 
overlap in the supports of the real and fake distributions. In 
this work, we propose the Multi-Scale Gradient Generative 
Adversarial Network (MSG-GAN), a simple but effective technique 
for addressing this by allowing the flow of
gradients from the discriminator to the generator at 
multiple scales. This technique provides a stable approach for
high resolution image synthesis, and serves as an alternative 
to the commonly used progressive growing technique.
We show that MSG-GAN converges stably on a variety of
image datasets of different sizes, resolutions and domains,
as well as different types of loss functions and architectures,
all with the same set of fixed hyperparameters. 
When compared to state-of-the-art GANs, our approach matches or
exceeds the performance in most of the cases we tried.

### Method overview

<p align="center">
<img alt="Architecture diagram" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/architecture_horizontal.jpg" />
<br>
</p>

Architecture of MSG-GAN, shown here on the base model proposed in 
ProGANs. Our architecture includes connections from the intermediate 
layers of the generator to the intermediate layers of the 
discriminator. Multi-scale images sent to the discriminator 
are concatenated with the corresponding activation volumes 
obtained from the main path of convolutional layers followed by a 
combine function (shown in yellow).

#### StyleGAN Modifications:
The MSG-StyleGAN model (in this repository) uses all the
modifications proposed by StyleGAN to the ProGANs architecture 
except the mixing regularization. Similar to 
MSG-ProGAN (diagram above), we use a 1 x 1 conv layer to obtain 
the RGB images output from every block of the StyleGAN generator 
leaving everything else (mapping network, non-traditional input and 
style adaIN) untouched. The discriminator architecture is same as 
the ProGANs (and consequently MSG-ProGAN) discriminator.

### System requirements

The code was built and tested for:
* 64-bit Python 3.6.7 
* TensorFlow 1.13.1 with GPU support.
* NVIDIA GPUs with at least 16GB of DRAM. We used variants of the Tesla V100 GPUs.
* NVIDIA driver 418.56, CUDA toolkit 10.1, cuDNN 7.3.1.

### How to run the code (Training)
Training can be run in the following 3 steps:

##### Step 1: Data formatting
The MSG-StyleGAN training pipeline expects the dataset to be in 
`tfrecord` format. This sped up the training to a great extent.
Use the `dataset_tool.py` tool to generate these tfrecords from your 
raw dataset. In order to use the tool, either select from the bunch 
of datasets that it already provides or use the `create_from_images`
option if you have a new dataset in the form of images. 
For full options and more information run:

    (your_virtual_env)$ python dataset_tool.py --help

##### Step 2: Run the training script
First step is to update the paths in the global configuration
located in `config.py`. For instance:
    
    """Global configuration."""
    
    # ----------------------------------------------------------------------------
    # Paths.
        
    result_dir = "/home/karnewar/self_research/msg-stylegan/"
    data_dir = "/media/datasets_external/"
    cache_dir = "/home/karnewar/self_research/msg-stylegan/cache"
    run_dir_ignore = ["results", "datasets", "cache"]
    
    # ----------------------------------------------------------------------------

The `result_dir` is where all the trained models, training logs and 
evaluation score logs will be reported. The `data_dir` should 
contain the different datasets used for training 
under separate subdirectories, while the `cache_dir` stores any 
repeatedly required objects in the training. For instance the
Mean and Std of the real images while calculating the FID.

Following this, download the inception net weights from 
[here](https://drive.google.com/uc?id=1MzTY44rLToO5APn8TZmfR7_ENSe5aZUn)
and place them in `result_dir + "/inception_network/inception_v3_features.pkl"`.

Finally, modify the configurations in the `train.py` as per your 
situation and start training by just running the `train.py` script.

    (your_virtual_env)$ python train.py

### Pretrained models

| Dataset        | Size  |Resolution| GPUs used   | FID score | Link            | 
| :---           | :---  |:--- | :---        | :---      | :---            |
| LSUN Churches  | ~150K |256 x 256| 8 V100-16GB | 5.20      | [drive link](https://drive.google.com/open?id=1_GuTi2-ZrY0A3kzAlxLFJdBgYHZRiSCt) |    
| Oxford Flowers | ~8K   |256 x 256| 2 V100-32GB | 19.60     | [drive link](https://drive.google.com/open?id=1oJDfaKvntxyhECp7eb1qPNmnjJek4R3d) |
| Indian Celebs  | ~3K   |256 x 256| 4 V100-32GB | 28.44     | [drive link](https://drive.google.com/open?id=1BflibIAA46KwRajKVE3hGo5g6qQ66uqO) |
| CelebA-HQ      | 30K   |1024 x 1024| 8 V100-16GB | 6.37      | [drive link](https://drive.google.com/open?id=1IP7J-a3HT7EcuHrb0Qz9fsA2CVTeDz08) |
| FFHQ           | 70K   |1024 x 1024| 4 V100-32GB | 5.80      | [drive link](https://drive.google.com/open?id=1hgykdbFRgIacGsAC4jlnUqckAY5lvB6t) |

### How to use pretrained models 
We provide three scripts `generate_multiscale_samples.py`, 
`generate_samples.py` and `latent_space_interpolation_video.py` 
which can be used to generate multi-scale generated images grids, 
highest resolution samples and latent space interpolation video 
respectively. Please see the below example.
    
    (your_virtual_env)$ python latent_space_interpolation_video.py \
    --pickle_file /home/karnewar/msg-stylegan/00004-msg-stylegan-visual_art-4gpu/best_model.pkl \
    --output_file /home/karnewar/msg-stylegan/visual_art_interpolation_hd.avi \
    --num_points 30 \
    --transition_points 30 \
    --resize 800 1920 \

### How to run evaluation
The training pipeline already computes the metric during training 
along with a tensorboard log. But, in case you wish to evaluate
the trained models again for research baseline (or for some other reason,
say fast training and separate evaluation), Please use the `run_metrics.py`
script. Modify the following lines according to your situation:

    tasks = []
    tasks += [
        EasyDict(
            run_func_name="run_metrics.run_pickle",
            network_pkl="/home/karnewar/msg-stylegan/00002-msg-stylegan-indian_celebs-4gpu/network-snapshot.pkl",
            dataset_args=EasyDict(tfrecord_dir="indian_celebs/tfrecords", shuffle_mb=0),
            mirror_augment=True,
        )
    ]  
    # tasks += [EasyDict(run_func_name='run_metrics.run_snapshot', run_id=100, snapshot=25000)]
    # tasks += [EasyDict(run_func_name='run_metrics.run_all_snapshots', run_id=100)]

    # How many GPUs to use?
    submit_config.num_gpus = 1
    # submit_config.num_gpus = 2
    # submit_config.num_gpus = 4
    # submit_config.num_gpus = 8

and run:
    
    (your_virtual_env)$ python run_metrics.py
    
The `run_snapshot` and `run_pickle` do practically the same thing with
the minor exception of the usage. The former needs a `snapshot_id` and `run_id`
and the files are located automatically, whereas the latter needs the pickle file
to be provided. I personally find the run_pickle much more useful. 
The `run_all_snapshots` function takes the `run_id` and evaluates 
all snapshots located in that run_dir.

### Stability and Ease of Use :)
Usually, it is the case that stability and easy usage are 
not the terms that you'd use in the context of a GAN :laughing:. 
But with the multi-scale gradients in the GAN, the training is quite 
stable. We show a juxtaposing experiment for this as follows:

<p align="center">
<img alt="progan_stability" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/Combined_1.jpg" width="49%" height="50%"/>
<img alt="msggan_stability" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/Combined_2.jpg" width="49%" height="50%"/>
<br>
</p>

We quantify the image stability during training. These plots show the MSE between images generated from the same latent code at
the beginning of sequential epochs (averaged over 36 latent samples) on the CelebA-HQ dataset. MSG-GAN converges
stably over time while Progressive Growing continues to vary significantly across epochs. Please note that 
the first half of the epochs are spent in fading in the new layer, but 
apparently, even for the subsequent epochs, the changes made are quite significant.

<p align="center">
<img alt="training_explanation" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/modified_synchronization.jpg" />
<br>
</p>

During training, all the layers in the MSG-GAN synchronize across the generated resolutions fairly early in the
training and subsequently improve the quality of the generated images at all scales simultaneously. Throughout the training
the generator makes only minimal incremental improvements to the images generated from fixed latent points.

### Qualitative examples

<p align="center">
<b> Simultaneous multi-scale latent space interpolation FFHQ [1024 x 1024] </b> <br>
<img alt="ffhq_multi_scale" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/cut_vid.gif" />
<br>
</p>

<p align="center">
<b> CelebA-HQ [1024 x 1024] </b> <br>
<img alt="CelebA-HQ" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/celebahq_1.jpg" />
<br>
</p>

<p align="center">
<b> FFHQ [1024 x 1024] </b> <br>
<img alt="FFHQ" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/ffhq_2.jpg" />
<br>
</p>

<p align="center">
<b> LSUN Churches [256 x 256] </b> <br>
<img alt="LSUN Churches" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/churches_3.jpg" />
<br>
</p>

<p align="center">
<b> Oxford Flowers [256 x 256] </b> <br>
<img alt="Oxford Flowers" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/flowers_3.jpg" />
<br>
</p>

<p align="center">
<b> Indian Celebs [256 x 256] </b> <br>
<img alt="Indian Celebs" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/ic_3.jpg" />
<br>
</p>

<p align="center">
<b> More Full resolution CelebA-HQ samples [1024x 1024] </b> <br>
<img alt="full_res_hq_sheet" src="https://github.com/akanimax/msg-stylegan-tf/blob/master/diagrams/full_res_hq_sheet.png" />
<br>
</p>

### Cite our work
    @article{karnewar2019msg,
      title={MSG-GAN: Multi-Scale Gradients for Generative Adversarial Networks},
      author={Karnewar, Animesh and Wang, Oliver},
      journal={arXiv preprint arXiv:1903.06048},
      year={2019}
    }

### Other contributors
Please feel free to open PRs here if 
you train on other datasets using this architecture. 
<br>

### Thanks and regards

#### \[:star: New :star:\] Please check out my new IG handle [@the_GANista](https://www.instagram.com/the_GANista/). I will be posting fun GAN based visual art here. :). 

Thank you all for supporting and encouraging my work. I hope
this will be useful for your research / project / work.

As always, any suggestion / feedback / contribution is always welcome :smile:.

Cheers :beers:! <br>
@akanimax :)
