# Setting up the environment
1. Go to project directory, type `bash`
2. In file `bash_script/run_train_magloss.sh`, copy line 14 + csh  
   ``` bash
   bsub -Is -q ML_GPU -app TensorFlow -P d_00993 -gpu num=1 -m GPU_3090 -J "LongJob" csh
   ```
   This command enters GPU environment: `bash`
3. Copy the commands in `bash_script/run_train_magloss.sh` to terminal and execute
4. Run the python files needed. E.g.
   ``` bash
   Python3 run_evaluation.py -i CSD_data/after_filter/gaming -output_data/ -m models/DTLN_model_72h_morenearend_resumefilterresumelr.h5  
   ```
---
# Model testing

## Test model using self-gen test audio
1. `filter/filter.py`  
    - [line 162] `enhance_pars`: change the first value from 0 to 3 (4 sets of results)
    - Execute the command: `python filter/filter.py`
2. `filter/test_filter.py`
    - Change the path of input and output audio accordingly
    - Execute the command: `python filter/test_filter.py`
    - Copy the output mic audio and input lpb audio to a same folder
    - Example usage:
      ```bash
      cp filter/output/float_float.wav CSD_data/after_filter/jingming_audio
      cp CSD_data /after_filter/15pb/linux_0/lpb.wav CSD_data/after_filter/jingming_audio
      cd CSD_data/after_filter/jingming_audio
      mv float_float.wav mic.wav
      ```
3. `calculate_cls3.py`
    - `python calculate_cls3.py`
    - Store `output_data/mic.wav`

---

# Model training
1. general sequenceL: train script  &rarr; model  &rarr; data
2. When continue the model training, need to load the trained model
    ```
    model_weight = [trained model]
    python3 train_loss.py -f True
    ```
## Filter Training
1. `python3 train_script/train-loss.py` (train.py &rarr; DTLN_model_loss.py  &rarr; data_gene)
2. dataset: data.gene_filter_save

## Filter + Classifier Training
1. resent  
    `python3 classifier/train_resnet.py` (train.py &rarr; resnet18.py &rarr; data_gene)
2. DTLN + classifier  
    `python3 calssifier/train_DTLN_resnet.py` (train.py &rarr; DTLN_model_3clasresnet.py &rarr; data_gene)
3. Double talk percentage (no filter)  
    `python3 train_script/train_loss.py` (train.py &rarr; DTLN_model_loss.py &rarr; data_gene)  
    dataset: data_gene_loss (adjust percentage of double talk)  
        1. line 419, 0.2 &rarr; 0.1  
        2. line 421 ~ 424, 0.2 &rarr; 0

## DTLN_model_noconv
1. model: `DTLN_model_noconv.py`
2. data: `data_gene_CSDproc.py`
3. change data path &rarr; `save_npz/18h_CSDprocnew_save_npz/72h_CSDprocnew`
4. `data_gene_save_IR.py`
5. train: change DTLN_model path

## DTLN_model_noconv
1. model: `DTLN_model_noconv.py`
2. data: `save_npz/18h_CSDprocnew_morenear`
3. train:   
    `python3 train.py -f False`   
    `python3 train.py -f True` (load model weights)

---
# Evaluation (PESQ, SNR)
## Calculate PESQ using synthetic dataset
### Input test audio  
AECchallenge_datasets/synthetic  
  - nearend_mic_signal  &rarr; mic.wav
  - farend_speech  &rarr; lpb.wav
  - nearend_speech  &rarr; label
### Steps
1. `python run_evaluation.py`  &rarr; audio after AEC
2. `cal_pesq.py`  
  ref  &rarr; label  
  deg  &rarr; model output


## Calculate PESQ for DTLN-aec
1. `pip install tensorflow-gpu==2.0.0`
2. `python run_aec.py`

## SNR
`calculate_snr_noprocess.py`
- clean_speech &rarr; label
- farend &rarr; lpb
- nearend &rarr; mic
