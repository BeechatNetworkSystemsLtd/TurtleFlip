# TurtleFlip: Fooling OCR systems through a black-box adversarial attack

# Summary

**Research goal**

Modify images to make the text on them unrecognizable by the Facebook/Instagram OCR system, but readable by humans.

**Conclusion**
- **We have demonstrated a successful adversarial attack on Instagram's OCR systems.**
- There was no ready-to-use solution
- There are two vectors of attack on the OCR systems: make text undetectable and make detected text unrecognizable
- OCR systems without image preprocessing are easy to fool
- OCR systems with a good image preprocessing are robust, even when images are not readable by humans
- When text is recognized with minor errors, humans still can read it, but it can become unreadable to machine natural language processing algorithms
- Best performance without image transformation is achieved with a small text, text with effects like a shadow, and fancy fonts such as script
- The most effective attacks are  are distortion, watermarks and combination of different transformation methods

# Research:

## Facebook OCR

Facebook made Rosetta that is a large scale machine learning system for text detection and recognition in images. It is closed source, but we have some information about it. 

It has pipeline of image preprocessing, text detection and text recognition algorithms. Pipeline looks similar to Google Cloud Vision, but implementation is different.

Facebook's Paper & System description:
[Rosetta: Understanding text in images and videos with machine learning - Facebook Engineering](https://engineering.fb.com/2018/09/11/ai-research/rosetta-understanding-text-in-images-and-videos-with-machine-learning/)

## Adversarial attacks

There are two types of attacks:

- **White box.** Used when we have source codes of the target model. Not relevant to us as we do not have an access to target OCR system.
- **Black box.** When we know nothing about the model and have no access to their code. In other words it is a case when we use a model as an API.

**Models**
|Name                            |GitHub                                                        |Type of Model|Language|Supported Framework       |Git Stars|Year|
|--------------------------------|--------------------------------------------------------------|-------------|--------|--------------------------|---------|----|
|cleverhans                      |https://github.com/cleverhans-lab/cleverhans                  |Both         |Python  |PyTorch, Tensorflow       |5000     |2021|
|adversarial-robustness-toolbox  |https://github.com/Trusted-AI/adversarial-robustness-toolbox  |Both         |Python  |Keras, PyTorch, Tensorflow|2000     |2021|
|foolbox                         |https://github.com/bethgelab/foolbox                          |Both         |Python  |PyTorch, Tensorflow       |1800     |2020|
|AdvBox                          |https://github.com/advboxes/AdvBox                            |White Box    |Python  |Keras, PyTorch, Tensorflow|1000     |2020|
|advertorch                      |https://github.com/BorealisAI/advertorch                      |White Box    |Python  |PyTorch                   |813      |2020|
|DeepRobust                      |https://github.com/DSE-MSU/DeepRobust                         |White Box    |Python  |PyTorch                   |360      |2021|
|Adversarial-Face-Attack         |https://github.com/ppwwyyxx/Adversarial-Face-Attack           |Black Box    |Python  |Tensorflow                |322      |2019|
|adversarial-attacks-pytorch     |https://github.com/Harry24k/adversarial-attacks-pytorch       |Both         |Python  |PyTorch                   |320      |2021|
|pytorch-cnn-adversarial-attacks |https://github.com/utkuozbulak/pytorch-cnn-adversarial-attacks|White Box    |Python  |PyTorch                   |310      |2019|
|adversarial-examples-pytorch    |https://github.com/sarathknv/adversarial-examples-pytorch     |White Box    |Python  |PyTorch                   |286      |2019|
|Non-Targeted-Adversarial-Attacks|https://github.com/dongyp13/Non-Targeted-Adversarial-Attacks  |White Box    |Python  |Tensorflow                |182      |2018|
|tree-ensemble-attack            |https://github.com/chong-z/tree-ensemble-attack               |White Box    |C++     |Custom                    |10       |2020|
|simple-blackbox-attack          |https://github.com/cg563/simple-blackbox-attack               |Black Box    |Python  |PyTorch                   |111      |2019|


Most of the open source adversarial attack models are related to the Image Classification problem (such as faces) and have white box approach. They are not relevant in our case.

There are also a couple papers for OCR adversarial attacks. Some are without source code, other are outdated and not yet effective.

**Therefore, we must implement our own black box OCR attack method.**

# Tests

The attacks have been tested against two well-known OCR systems on three images using many image transformation methods.

### OCR systems

- **Google Cloud Vision.** One of the best OCR solutions on the market. It does OCR, as well as text preprocessing, and even can recognize handwritten text.
- **Google Tesseract.** A well-known OCR tool by Google. Works well with scanned documents. For more complex cases requires image preprocessing, like: text detection, text binarization, noise reduction, etc., which is not part of Tesseract itself.

[Source images](https://www.notion.so/992068aac7054984a6e26976bda41181)

### Image transformation attack methods

- Strike
- Noise
- JPEG Effect
- Scale
- Skew
- Aspect ratio
- Rotation
- Blur
- Distortion
- Clone stamping
- Watermark
- Color inversion
- Image overlay
- Combination of different methods

### Evaluation

Google Cloud Vision results are labeled with 3 different tags:

- **Good.** No or minor mistakes. Most of the text is both human and machine readable.
- **Average.** Recognition is not perfect. It can be read by humans, but machines may not understand the meaning of words.
- **Bad.** Text is unreadable.

Tesseract results were evaluated using the [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) metric, which shows how many characters differ in two compared texts. The lower the number, the better the match, the better the OCR.

## Output

No transformation: Good detection

**Tests results on different transformation attacks:**
1.Strike: Good detection
2.a. Medium gaussian noise: Good detection
2.b. Strong gaussian noise: Average detection
2.c. Pepper noise: Good detection
2.d. Salt noise: Good detection
2.e. Poisson noise: Good detection
2.f. Procedural noise: Average detection
3.JPEG effect: Good to average detection
4.a. Medium scale: Average detection
4.b. Low scale: Bad to average detection
5.a. Medium blur: Average detection
5.b. Strong blur: Bad detection
6.Skew: Average detection
7.a. Medium aspect ratio height: Average detection
7.b. Low aspect ratio height: Bad to average detection
7.c. Medium aspect ratio width: Good to average detection
7.d. Low aspect ratio width: Bad to average detection
8.a. Rotation 90º CCW: Good detection
8.b. Rotation 90º CW: Good detection
8.c. Rotation 180º: Good detection
9.a. Medium distortion: Bad to average detection
9.b. Strong distortion: Bad detection
10 Clone stamping: Average to bad detection
11 Watermark: Average to good detection
12 Color Inversion: Good detection
13 Image overlay: Good detection
**14. a. Combination of strike and noise: Bad to average detection**
**14. b. Combination of medium distortion and gaussian noise: Bad to average detection**

We have highlighted the transformation attacks that worked well against Google Vision and were subjectively easier to read for the developing team. From all the tests we can conclude cloud vision fails the most with Ruled Surface Distortion combined with Random Noise Distortions.

With this hypothesis in mind, we move onto the next phase of research, testing an image that triggers the following notification from Instagram when posted:

![ImageDetected](http://176.58.110.19/wp-content/uploads/2021/02/detected.png)

We use the Jupyter Notebook called ```test-degrade.ipynb``` of [NVlabs/ocrodeg GitHub Repository](https://github.com/NVlabs/ocrodeg) to process the test image and see if Instagram detects information and posts a notification regarding government vaccine sources to confirm a successful adversarial attack.

**Test results:**

Random Noise Distortion @ 0.75
Ruled Surface Distortion @ 120

![ImageUndetected](http://176.58.110.19/wp-content/uploads/2021/02/undetected.png)

The next step will be converting the Jupyter Notebook into a script, and then implementing a simple GUI to load an image and being able to export it; ie: making a simple GUI Python app.

This system is licensed under the MIT License.
