# How to Run Stable Diffusion Locally to Generate Images
Following in the footsteps of [DALL-E 2](https://www.assemblyai.com/blog/how-dall-e-2-actually-works/) and [Imagen](https://www.assemblyai.com/blog/how-imagen-actually-works/), the new Deep Learning model **[Stable Diffusion](https://github.com/CompVis/stable-diffusion?ref=assemblyai.com)** signifies a quantum leap forward in the text-to-image domain. Released earlier this month, Stable Diffusion promises to democratize text-conditional image generation by being efficient enough to run on consumer-grade GPUs. Just this Monday, Stable Diffusion checkpoints were released for the first time, meaning that, _right now_, you can generate images like the ones below with just a few words and a few minutes time.

![](https://www.assemblyai.com/blog/content/images/2022/08/grid-1.png)

**This article will show you how to install and run Stable Diffusion**, both on [GPU](#how-to-install-stable-diffusion-gpu) and [CPU](#how-to-install-stable-diffusion-cpu), so you can get started generating your own images. Let's dive in!

[#](#use-stable-diffusion-in-colab)Use Stable Diffusion in Colab
----------------------------------------------------------------

Before we look at how to install and run Stable Diffusion locally, you can check out the below Colab notebook in order to see how to use Stable Diffusion non-locally. **Note that you will need Colab Pro in order to generate new images** given that the free version of Colab has slightly too little VRAM for sampling.

[Stable Diffusion in Colab (GPU)](https://colab.research.google.com/drive/1uWCe41_BSRip4y4nlcB8ESQgKtr5BfrN?usp=sharing&ref=assemblyai.com)

**You can also run Stable Diffusion on CPU in Colab** if you do not have Colab Pro, but note that image generation will take a relatively long time (8-12 minutes):

[Stable Diffusion in Colab (CPU)](https://colab.research.google.com/drive/1NA8XytvxOT841JtaTao896Gm_SFT66N4?usp=sharing&ref=assemblyai.com)

You can also check out our [Stable Diffusion Tutorial](https://www.youtube.com/watch?v=CtlxYoya2Sc&ref=assemblyai.com) on YouTube for a walkthrough of using the GPU notebook.

[#](#how-to-install-stable-diffusion-gpu)How to Install Stable Diffusion (GPU)
------------------------------------------------------------------------------

You will need a UNIX-based operating system to follow along with this tutorial, so if you have a Windows machine, consider using a [virtual machine](https://www.vmware.com/?ref=assemblyai.com) or [WSL2](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10?ref=assemblyai.com#1-overview).

### Step 1: Install Python

First, check that Python is installed on your system by typing `python --version` into the terminal. If a Python version is returned, continue on to the [next step](#step-2-install-miniconda). Otherwise, install Python with

```null
sudo apt-get update
yes | sudo apt-get install python3.8
```

### Step 2: Install Miniconda

Next, we need to ensure the package/environment manager [conda](https://docs.conda.io/en/latest/?ref=assemblyai.com) is installed. Enter `conda --version` in the terminal. If a conda version is returned, move on to the [next step](#step-3-clone-the-stable-diffusion-repository).

Otherwise, go to the [conda website](https://docs.conda.io/en/latest/miniconda.html?ref=assemblyai.com) and download and run the appropriate Miniconda installer for your version of Python and operating system. For Python3.8, you can download and run the installer with the following commands:

```null
wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-x86_64.sh
bash Miniconda3-py38_4.12.0-Linux-x86_64.sh
```

Hold down _Enter_ to get through license and then type "yes" to continue when prompted. Next, press _Enter_ to confirm the installation location, and then type "yes" when asked if the installer should initialize Miniconda. Finally, close the terminal and then open a new one where you want to install Stable Diffusion.

### Step 3: Clone the Stable Diffusion Repository

Now we need to clone the Stable Diffusion repository. In the terminal, execute the following commands:

```null
git clone https://github.com/CompVis/stable-diffusion.git
cd stable-diffusion/
```

If you do not have git, you will need to install it with `sudo apt install git`. Make sure to read and accept the [Stable Diffusion license](https://github.com/CompVis/stable-diffusion/blob/main/LICENSE?ref=assemblyai.com) before cloning the repository.

### Step 4: Create Conda Environment

Next up we need to create the conda environment that houses all of the packages we'll need to run Stable Diffusion. Execute the below commands to create and activate this environment, named `ldm`

```null
conda env create -f environment.yaml
conda activate ldm
```

### Step 5: Download Stable Diffusion Weights

Now that we are working in the appropriate environment to use Stable Diffusion, we need to download the weights we'll need to run it. If you haven't already read and accepted the [Stable Diffusion license](https://github.com/CompVis/stable-diffusion/blob/main/LICENSE?ref=assemblyai.com), make sure to do so now. Several Stable Diffusion checkpoint versions have been released. Higher version numbers have been trained on more data and are, in general, better performing than lower version numbers. We will be using **checkpoint v1.4**. Download the weights with the following command:

```null
curl https://f004.backblazeb2.com/file/aai-blog-files/sd-v1-4.ckpt > sd-v1-4.ckpt
```

That's all of the setup we need to start using Stable Diffusion! Read on to learn how to generate images using the model.

[#](#how-to-generate-images-with-stable-diffusion-gpu)How to Generate Images with Stable Diffusion (GPU)
--------------------------------------------------------------------------------------------------------

To generate images with Stable Diffusion, open a terminal and navigate into the `stable-diffusion` directory. Make sure you are in the proper environment by executing the command `conda activate ldm`.

To generate an image, run the following command:

```null
python scripts/txt2img.py --prompt "YOUR-PROMPT-HERE" --plms --ckpt sd-v1-4.ckpt --skip_grid --n_samples 1

```

Troubleshooting

*   If you receive an `ImportError`, you may have to run `sudo apt-get install libsm6 libxrender1 libfontconfig1`
*   If execution suddenly stops and `killed` is printed to the terminal, you are likely running into an out-of-memory error. Run `cat /var/log/kern.log` to see useful logs.
*   If you run into a `RuntimeError: CUDA out of memory` error, try cutting down the size of the image (and make sure you are only sampling one image with `--n_samples 1`). The minimum image size is 256x256.

Where you replace `YOUR-PROMPT-HERE` with the caption for which to generate an image (leaving the quotation marks). Running this command with the prompt _**"a photorealistic vaporwave image of a lizard riding a snowboard through space"**_ outputs the following image:

![](https://www.assemblyai.com/blog/content/images/2022/08/lizard.png)

The above image was generated in about a minute using an Ubuntu 18.04 VM in GCP with a NVIDIA Tesla K80.

### Script Options

You can customize this script with several command-line arguments to tailor the results to what you want. Let's take a look some that might come in handy:

1.  `--prompt` followed by a sentence in quotation marks will specify the **prompt** to generate the image for. The default is "_a painting of a virus monster playing guitar_".
2.  `--from-file` specifies a filepath for a **file of prompts** to use to generate images for.
3.  `--ckpt` followed by a path specifies which **checkpoint** of model to use. The default is `models/ldm/stable-diffusion-v1/model.ckpt`.
4.  `--outdir` followed by a path will specify the **output directory** to save the generate image to. The default is `outputs/txt2img-samples`.
5.  `--skip_grid` will skip creating an image that combines.
6.  `--ddim_steps` followed by an integer specifies the **number of sampling steps** in the [Diffusion process](https://www.assemblyai.com/blog/diffusion-models-for-machine-learning-introduction/#diffusion-modelsintroduction). Increasing this number will increase computation time but may improve results. The default value is 50.
7.  `--n_samples` followed by an integer specifies **how many samples to produce** for each given prompt (the batch size). The default value is 3.
8.  `--n_iter` followed by an integer specifies **how many times to run the sampling loop**. Effectively the same as `--n_samples`, but use this instead if running into OOM error. See the [source code](https://github.com/CompVis/stable-diffusion/blob/69ae4b35e0a0f6ee1af8bb9a5d0016ccb27e36dc/scripts/txt2img.py?ref=assemblyai.com#L286) for clarification. The default value is 2.
9.  `--H` followed by an integer specifies the **height** of the generated images (in pixels). The default value is 512.
10.  `--W` followed by an integer specifies the **width** of generated images (in pixels). The default value is 512.
11.  `--scale` followed by a float specifies the [**guidance scale**](https://www.assemblyai.com/blog/how-imagen-actually-works/#classifier-free-guidance) to use. The default value is 7.5
12.  `--seed` followed by an integer allows for setting the **random seed** (for reproducible results). The default value is 42.

You can see a full list of possible arguments with default values in the `[txt2img.py](https://github.com/CompVis/stable-diffusion/blob/7b8c883b078024f68b56e862f247a64f9e282aac/scripts/txt2img.py?ref=assemblyai.com#L45)` file. Let's see a more complicated generation prompt using these optional arguments now.

In the `stable-diffusion` directory, create a file called `prompts.txt`. Create several prompts, one on each line of the file. For example:

![](https://www.assemblyai.com/blog/content/images/2022/08/image-5.png)

Now, in `stable-diffusion` directory in the terminal, run

```null
python scripts/txt2img.py \
--from-file prompts.txt \
--ckpt sd-v1-4.ckpt \
--outdir generated-images \
--skip_grid \
--ddim_steps 100 \
--n_iter 3 \
--H 256 \
--W 512 \
--n_samples 3 \
--scale 8.0 \
--seed 119
```

Two resulting images for each caption can be seen below. The above command is intended to serve as an example of using more command-line arguments, and not as an example of optimal arguments. In general, larger images are empirically found to be both higher quality and have greater image/caption similarity, and a lower guidance scale would likely yield better results. Continue on to the [next section](#tips-and-tricks) for more information about improving Stable Diffusion results.

![](https://www.assemblyai.com/blog/content/images/2022/08/grid.png)

[#](#how-to-install-stable-diffusion-cpu)How to Install Stable Diffusion (CPU)
------------------------------------------------------------------------------

### Step 1: Install Python

First, check that Python is installed on your system by typing `python --version` into the terminal. If a Python version is returned, continue on to the next step. Otherwise, install Python with

```null
sudo apt-get update
yes | sudo apt-get install python3.8
```

### Step 2: Download the Repository

Now we need to clone the Stable Diffusion repository. We will be using [a fork](https://github.com/bes-dev/stable_diffusion.openvino?ref=assemblyai.com) that can accommodate CPU inference. In the terminal, execute the following commands:

```null
git clone https://github.com/bes-dev/stable_diffusion.openvino.git
cd stable_diffusion.openvino
```

If you do not have git, you will need to install it with `sudo apt install git`. Make sure to read and accept the [Stable Diffusion license](https://github.com/CompVis/stable-diffusion/blob/main/LICENSE?ref=assemblyai.com) before cloning the repository.

### Step 3: Install Requirements

Install all necessary requirements with

```null
pip install -r requirements.txt

```

Note that Scipy version 1.9.0 is a listed requirement, but it is incompatible with older versions of python. You may need to change the Scipy version by editing `requirements.txt` to read e.g. `scipy==1.7.3` before runnning the above command.

### Step 4: Download Stable Diffusion Weights

Now that we are working in the appropriate environment to use Stable Diffusion, we need to download the weights we'll need to run it. If you haven't already read and accepted the [Stable Diffusion license](https://github.com/CompVis/stable-diffusion/blob/main/LICENSE?ref=assemblyai.com), make sure to do so now. Several Stable Diffusion checkpoint versions have been released. Higher version numbers have been trained on more data and are, in general, better performing than lower version numbers. We will be using **checkpoint v1.4**. Download the weights with the following command:

```null
curl https://f004.backblazeb2.com/file/aai-blog-files/sd-v1-4.ckpt > sd-v1-4.ckpt
```

That's all of the setup we need to start using Stable Diffusion! Read on to learn how to generate images using the model.

[#](#how-to-generate-images-with-stable-diffusion-cpu)How to Generate Images with Stable Diffusion (CPU)
--------------------------------------------------------------------------------------------------------

Now that everything is installed, we are prepared to generate images with Stable Diffusion. **To generate an image, simply run the below command**, changing the prompt to whatever you want.

```null
python demo.py --prompt "bright beautiful solarpunk landscape, photorealism"
```

The inference time will be high at around **8-12 minutes**, so feel free to grab a cup of coffee while Stable Diffusion runs. Below we can see the output from running the above command:

![](https://www.assemblyai.com/blog/content/images/2022/09/image-24.png)

[#](#tips-and-tricks)Tips and Tricks
------------------------------------

While you're getting started using Stable Diffusion, keep these tips and tricks in mind as you explore.

### Prompt Engineering

The results from a text-to-image model can be sensitive to the phrasing that is used to describe the desired scene. **Prompt Engineering** is the practice of tailoring the prompt to acquire desired results. For example, if low-quality images are being generated, try prepending the caption with "an image of". You can also specify different styles and mediums in order to achieve different effects. Check out each of the below the below drop-downs for ideas:

Image Types

Try prepending the caption with one of the following for different effects:

*   "An image of"
*   "A photograph of"
*   "A headshot of"
*   "A painting of"
*   "A vision of"
*   "A depiction of"
*   "A cartoon of"
*   "A drawing of"
*   "A figure of"
*   "An illustration of"
*   "A sketch of"
*   "A portrayal of"

Styles

You can specify different styles to achieve different results. Try adding one or more of the below adjectives to a prompt and observing the effects.

*   "Modernist(ic)"
*   "Abstract"
*   "Impressionist(ic)"
*   "Expressionist(ic)"
*   "Surrealist(ic)"

Aesthetics

You can experiment with specifying different aesthetics as well. Try adding one or more of the below adjectives to a prompt and observing the effects.

*   "Vaporwave"
*   "Synthwave"
*   "Cyberpunk"
*   "Solarpunk"
*   "Steampunk"
*   "Cottagecore"
*   "Angelcore"
*   "Aliencore"

Artists

You can even try specifying different artists to achieve different visual results. Try appending one of the following to a prompt:

*   Painters:
    *   "in the style of Vincent van Gogh"
    *   "in the style of Pablo Picasso"
    *   "in the style of Andrew Warhol"
    *   "in the style of Frida Kahlo"
    *   "in the style of Jackson Pollock"
    *   "in the style of Salvador Dali"
*   Sculptors:
    *   "in the style of Michelangelo"
    *   "in the style of Donatello"
    *   "in the style of Auguste Rodin"
    *   "in the style of Richard Serra"
    *   "in the style of Henry Moore"
*   Architects:
    *   "in the style of Frank Lloyd Wright"
    *   "in the style of Mies van der Rohe"
    *   "in the style of Eero Saarinen"
    *   "in the style of Antoni Gaudi"
    *   "in the style of Frank Gehry"

### Tuning Sampling Parameters

When tuning sampling parameters, you can utilize the below empirical observations to guide your exploration.

#### Image Size

In general, it appears empirically that larger images are much better in both image quality and caption alignment than smaller images. See the below examples for the prompt _"Guy Fieri giving a tour of a haunted house"_ for both 256x256 and 512x512 sized images:

![](https://www.assemblyai.com/blog/content/images/2022/08/fieri.png)

256x256 vs. 512x512 sample comparison for the prompt "Guy Fieri giving a tour of a haunted house"

#### Number of Diffusion Steps

It appears that the number of steps in the diffusion process does not affect results much _beyond_ a certain threshold of about 50 timesteps. The below images were generated using the same random seed and prompt _**"A red sports car"**_. It can be seen that a greater number of timesteps consistently improves the quality of the generated images, but past 50 timesteps improvements are only manifested in a slight change to the incidental environment of the object of interest. The details of the car are in fact almost fully consistent from 25 timesteps onward, and it is the environment that is improving to become more appropriate for the car in greater timesteps.

![](https://www.assemblyai.com/blog/content/images/2022/08/comparison.png)

[full resolution version](https://raw.githubusercontent.com/AssemblyAI-Examples/stable-diffusion-tutorial/main/comparison.png?ref=assemblyai.com)

#### Image Aspect Ratio

It appears that image quality and caption similarity as a function of aspect ratio depend on the input caption. The below images have the same area but different aspect ratios, all generated using the caption _**"A steel and glass modern building"**_. The results are relatively uniform, although the vertical image appears to be the best, followed by square and then horizontal. This should be unsurprising given that modern buildings of this type are tall and skinny. Performance as a function of aspect ratio therefore seems to be subject-dependent.

![](https://www.assemblyai.com/blog/content/images/2022/08/aspect.png)

Unfortunately, Stable Diffusion is limited to factorizable aspect ratios, making more finely grained experiments impossible, but square images should suffice for most purposes anyway.

### Checkpoint Symbolic Link

To avoid having to suppling the checkpoint with `--ckpt sd-v1-4.ckpt` each time you generate an image, you can create a symbolic link between the checkpoint and the default value of `--ckpt`. In the terminal, navigate to the `stable-diffusion` directory and execute the following commands:

```null
mkdir -p models/ldm/stable-diffusion-v1/
ln -s sd-v1-4.ckpt models/ldm/stable-diffusion-v1/model.ckpt 
```

Alternatively, you can simply move the checkpoint into the default `--ckpt` location:

```null
mv sd-v1-4.ckpt models/ldm/stable-diffusion-v1/model.ckpt
```

### Did I Really Get Rick Rolled?

Yes. If Stable Diffusion detects that a generated image may violate its [safety filter](https://stability.ai/blog/stable-diffusion-public-release?ref=assemblyai.com), the generated image will be replaced with a still of Rick Astley.

![](https://www.assemblyai.com/blog/content/images/2022/08/00002-1.png)

[#](#final-words)Final Words
----------------------------

That's all it takes to generate images using the new Stable Diffusion model - don't forget to share your fun creations with us on [Twitter](https://twitter.com/AssemblyAI?ref=assemblyai.com)! If you want to learn more about how Stable Diffusion works, you can check out our [Introduction to Diffusion Models for Machine Learning](https://www.assemblyai.com/blog/diffusion-models-for-machine-learning-introduction/) article. If you enjoyed this article, feel free to check out more of our [blog](https://www.assemblyai.com/blog/) or [YouTube](https://www.youtube.com/c/AssemblyAI?ref=assemblyai.com) channel for Machine Learning content, or feel free to follow our newsletter to stay in the loop for new releases.

Some images we generated using Stable Diffusion:

![](https://www.assemblyai.com/blog/content/images/2022/08/00000-1.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00008.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00042.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00065.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00064.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00063.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00046-1.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00071.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00072.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00003-1.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00008-1.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00010.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00015.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00024.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00031-1.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00032.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00034-1.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00040.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00044.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00047.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00052-1.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00053.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00058.png)

![](https://www.assemblyai.com/blog/content/images/2022/08/00059.png)