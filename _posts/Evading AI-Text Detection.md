# Humanizing Machine-Generated Content: Evading AI-Text Detection through Adversarial Attack
*Ying Zhou, Ben He, Le Sun*


TL;DR

AI-text detectors can be fooled into classifying machine-generated text as human-written content using adversarial attacks.

### Importance of detecting AI generated content
- Detecting the spread of false information.
- Protection of intellectual property.
- Prevention of academic plagiarism.

### Current detection methods
1. Using statistical measures  
Statistical measures such as entropy,perplexity, and log-likelihood are used to determine whether content is AI generated.
For example DetectGPT by [Mitchell et al.(2023)](https://arxiv.org/pdf/2301.11305.pdf) defines a curvature-based criteria to detect AI-text based on the observation that LLM generated text often found in the negative curvature region of the model's log probability function.

2. Using neural classifiers.
A neural network is trained on labeled data to create a binary classifier. This apprach is prone to challenges
related to weak generalization and limited robustness against attacks.
3. Using Watermarking methods.

The watermarks are imperceptible patterns added to the AI-generated text. Watermarking can be done by randomly partitioning the vocabulary into a green list and a red list during text generation by dividing based on the hash of previously generated tokens [(Kirchenbauer et al, 2023)](https://proceedings.mlr.press/v202/kirchenbauer23a/kirchenbauer23a.pdf)].


## Adversarial Attack
The authors propose Adversarial Detection Attack on AI-Text (ADAT) which perturbs AI-generated text in a semantically preserving manner, while influencing the detector to classify machine-generated text as human generated content.

The paper considered white-box attacks and decision-based blackbox attacks.
In the white-box attack scenario, the attacker accesses comprehensive information about the detector model, its parameters, gradients, and training data.
In a black-box attack the attacker can only access the model's output; decision-based black box attack is where the attacker only has access to only the final decision of the model and not the rankings of all the outputs.  
The authors also created dynamic attacks to iteratively optimize
the interaction between the attacker and detector across multiple rounds.

### Algorithm Overview. 
The HMGC framework we introduce can be conceptualized as an ongoing interaction between the attacker and the detector.When presented with machine-generated text, the attacker iteratively modifies the text in an attempt to fool the detector. This process continues until the detector finally classifies the adversarial text as human-generated. 

The attacker in our HMGC framework comprises
four key concepts: the surrogate detection model Dθ, the word importance ranker R, the encoder based word swapper M, and a set of constraints C = {c1, c2, ..., ck}. 
#### Algorithm

procedure 1:Train Surrogate Model 

For each entry in the dataset get a predictio from the original model. Use the predictions from the original model to train the surrogate model. 

procedure 2. Pre-attack Assessment

Predict whether the current sample is humanwritten using the surrogate model.

procedure 3. Word Importance Ranking

Wcurr = {w1, w2, ..., wn} Split current sample to words and calculate word importance based on gradient and perplexity for each word. Sort Wcurr based on importance.

procedure 4. Mask Word Substitution

for wi n W[: k] do 

    Replace wi in tcurr with MASK token
    Obtain M candidates to fill in for tcurr
        if Dθ(topt) < Dθ(tcurr) then
            tcurr ← topt
        end if
    Post-checking for tcurr on a set of attack constraints C
end for

 procedure 5. Post-attack Checking

if Dθ(tcurr) < τ or reach attack limits 
    
    return tadv = tcurr 
    ▷ Algorithm terminates

else
    
    go to procedure 3. 
▷ Repeat the process

end if

## Details on Components.
Surrogate Victim Model

The surrogate model is trained based on RoBERTa with the binary classification task and emulates the behavior of the original detection model providing gradients for adversarial attacks which are used to compute the importance of words. 
The training supervision signal is distilled directly from the predictions of the original model. Formally, leveraging a pre-collected training dataset T, we employ the original detection model Dori to predict each sample, obtaining a set of prediction results.

Word Importance Ranking

Perturbing important words is more effective because the detector is more sensitive to individual words within the text.
The word importance ranking algorithm combines model gradients and perplexity derived from large language models.
Attacks are more effective on tokens with higher gradients on the victim model, whereas higher gradients indicate greater impacts on the final result. Consequently, we consider the gradient norm value corresponding to the
i-th token as the first aspect of word importance:

Language perplexity has been observed to be a key indicator for distinguishing between human and machine-generated text such that AI-Text has lower perplexity. The researchers introduced additional constraints to increase the
perplexity of the adversarial results, calculating the perplexity
importance as the difference in language perplexity before and after the i-th token is removed for each input token,.

Dynamic Detector Finetuning
After each round of attacks that
gather a substantial collection of adversarial samples, the researchers  train the surrogate
model. This process is designed to strengthen the detector’s defense capabilities against one specific form of attack, simulating a real-world application scenario where the detector accumulates user queries and continually enhances its capabilities.










