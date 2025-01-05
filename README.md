# Shadowcast: Stealthy Data Poisoning Attacks against Vision-Language Models

This study introduces Shadowcast, a stealthy data poisoning attack targeting Vision-Language Models (VLMs), making it vulnerable to manipulation with visually indistinguishable poisoned images. This study demonstrates two types of data poisoining attacks: a Label Attack, which causes VLMs to misclassify identities (e.g., confusing Donald Trump with Joe Biden), and a Persuasion Attack, which generates convincing but false narratives, like promoting junk food as healthy.

![Shadowcast](shadowcast_fig1.png)

Shadowcast is the first data poisoning attack against VLMs, which manipulates model responses to benign prompts. Unlike traditional poisoning attacks, Shadowcast creates stealthy, visually matching image/text pairs that are undetectable by humans. It works by subtly altering images to resemble a target concept and pairing them with texts that clearly convey the manipulated concept. This attack is effective even with a small number of poisoned samples and can manipulate VLMs in black-box settings. Shadowcast poses a more pervasive threat than jailbreaking attacks, as it impacts general users by tampering with training data rather than specific adversarial prompts.

## Attacker's obejective
The objective is to manipulate the model so that it generates text that misinterprets images from one concept (the original concept, denoted as C<sub>o</sub>) to another concept (the destination concept, denoted as C<sub>d</sub>). This study considers the following two kinds of attacks, each targeting a distinct type of destination concept C<sub>d</sub>.

1. **Label Attack:** The destination concept C<sub>d</sub> is a class label. The attacker’s objective is to
 manipulate the model so that when it encounters an image from the original concept C<sub>o</sub> (e.g., Donald
 Trump), it generates responses that mistake it for a different class C<sub>d</sub> (e.g., Joe Biden).

 2. **Persuasion Attack:** In this case, the destination concept C<sub>d</sub> is an elaborate narrative,
 different from the original concept C<sub>o</sub>. In Persuasion Attack, C<sub>d</sub> can involve more elaborate textual descriptions, fully utilizing
 the text generation capabilities of VLMs to create conceptually skewed narratives. For instance, a
 model subjected to Persuasion Attack might encounter an image representing ‘junk food’ (C<sub>o</sub>) and be
 manipulated to describe it as ‘healthy food rich in nutrients’ (C<sub>d</sub>). 

## Attacker's knowledge

This study uses both grey-box and black-box scenarios. 
1. In the grey-box scenarios, Shadowcast only requires access to the VLM’s vision encoder. While the grey-box assumption
is less feasible for closed-source VLMs, it remains relevant due to the prevalent use of open-source VLMs and vision encoders in various applications.

2. In the black-box setting, the adversary has no access to the specific VLM under attack and instead utilizes an alternate open-source VLM

## Attacker's capabilities

This study assume that the attacker 4 types of capabilities.

1. Attacker can inject a certain amount of poison data
(image/text pairs) into the model’s training dataset

2. Attacker has access to images representing both the
original and destination concepts (e.g., sourced from existing datasets or the internet)

3. Attacker has no control over the model during or after the training stage

4.  Attacker is limited to injecting poison samples,
consisting of image/text pairs, where each image appears benign and aligns with its corresponding text. This “clean-label” attack setting is in contrast to the “dirty-label” setting found in prior work on poisoning multimodal models. In the “dirty-label” setting, the poison samples comprise mismatched image/text pairs, which makes them more easily detectable through human inspection.

## Model training

In this study, the authors focus on the visual instruction tuning setting, where pretrained Vision-Language Models (VLMs) are further fine-tuned using high-quality datasets designed for visual instruction-following tasks.

## Overview of the Shadowcast

![Shadowcast](shadowcast_fig2.png)

As the attacker has access to collections of images {x<sub>o</sub>} and {x<sub>d</sub>}, representing the original
concept C<sub>o</sub> and the destination concept C<sub>d</sub>. The attacker’s goal is to manipulate the model into
responding to images x<sub>o</sub> with texts consistent with C<sub>d</sub>, using stealthy poison samples that can escape human visual inspection.
Shadowcast constructs congruent
image/text pairs as poison samples, illustrated in Figure 2. For text generation, Shadowcast carefully craft texts t<sub>d</sub> associated with the destination concept C<sub>d</sub> from clean images x<sub>d</sub>.
For image perturbation, Shadowcast introduces imperceptible perturbation to each clean image x<sub>d</sub> to obtain x<sub>p</sub>, which is close to an image x<sub>o</sub> from the original concept C<sub>o</sub> in the latent feature space.

Given that x<sub>p</sub> and x<sub>d</sub> are visually indistinguishable, the image/text pair (x<sub>p</sub>,t<sub>d</sub>) is visually congruent. During the training on poison samples, the VLM is trained to associate the representation of x<sub>p</sub> with t<sub>d</sub>. Since x<sub>p</sub> and x<sub>o</sub> are close in the latent feature space, the VLM consequently begins to associate the representation of x<sub>o</sub> with t<sub>d</sub>, effectively achieving the attacker’s goal

## Crafting the texts

To avoid human detection while steering VLMs towards the destination concept C<sub>d</sub> using minimal poison samples, the texts t<sub>d</sub> must adhere to visual consistency with images {x<sub>d</sub>} and consistently emphasize
the concept C<sub>d</sub> to maintian the concept consistency. To meet these two criteria, this study generate t<sub>d</sub> by first producing captions of images {x<sub>d</sub>} and then refining the captions using a language model, with specifics detailed below.

1. **Generating captions:** This study use an off-the-shelf VLM to generate a caption t<sub>caption</sub> for the image x<sub>d</sub> using the instruction “describe the image in details.” This step ensures that the caption
t<sub>caption</sub> matches the content in the image x<sub>d</sub>. However, even though x<sub>d</sub> is from the concept C<sub>d</sub>, it is possible that the caption t<sub>caption</sub> does not clearly convey the concept C<sub>d</sub>.  For example, when C<sub>d</sub> is “healthy food with various nutrition” and x<sub>d</sub> is a photo of a nutritious meal, the caption might only include descriptions of the food without
mentioning anything related to healthiness.

2. **Refining captions:** To obtain the text t<sub>d</sub> that clearly conveys and emphasizes the concept C<sub>d</sub>, this study use an LLM (e.g., GPT-3.5-turbo) to paraphrase the caption t<sub>caption</sub> with the explicit instruction to
emphasize the concept C<sub>d</sub> clearly.
    
    2.1. **C<sub>d</sub> is a label:** As an example, this study use “Joe Biden” as the destination concept C<sub>d</sub>. It can use the following instruction for paraphrasing the caption: “Paraphrase the following sentences to mention ‘Joe Biden’ in the response: ”

    2.2. **C<sub>d</sub> is a description** As an example, we use “healthy food with various nutrition” as C<sub>d</sub>. We use
    the following instruction: “Paraphrase the following sentences with the following requirements: (1)
    mention ‘healthy food’ in the response; (2) explain why the food in the sentences is healthy; If
    appropriate, mention how the food is rich in protein, essential amino acids, vitamins and fiber: ”

## Crafting the poison images

To craft the poison images {x<sub>p</sub>} for the visually matching poison samples {x<sub>p</sub>,t<sub>d</sub>}, it is important that each poison image x<sub>p</sub> visually resembles x<sub>d</sub> and is similar to an image x<sub>o</sub> of the concept C<sub>o</sub> in the latent feature space. 
This study applies the following objective for crafting poison images:

<img src="https://latex.codecogs.com/svg.latex?\color{red}\min_{x_p}%20\|F(x_p)%20-%20F(x_o)\|_2,%20\quad%20\text{s.t.}%20\quad%20\|x_p%20-%20x_d\|_\infty%20\leq%20\epsilon" />

Where F(·) is the vision encoder of the VLM that the attacker has access to, and ϵ is the perturbation
 budget.

## Conclusion
This study introduces the first VLM poisoning attack Shadowcast, which simultaneously causes
 pervasive impact on everyday, benign user prompts, avoids human inspection and subtly disseminates
 misinformation using coherent free-form texts. Furthermore, experiments of this study demonstrate that
 Shadowcast is effective across different VLM architectures and prompts, and is resilient to image
 augmentation and compression, proving its efficacy under realistic conditions.
 This work exposes new and practical vulnerabilities in VLMs.
## Limitation
 A limitation of this work is that they do not explored defense strategies against VLM poisoning
 attacks, an essential area for future research. Adapting strategies like filtering and
 adversarial training from defense methods used image classification presents unique challenges for VLMs, including compatibility with specific loss functions and architectures, high computational demands of VLMs, and potential reduction in model performance. Overcoming these challenges is vital for the responsible deployment of VLMs.
