# O: Hardware & Limitations meet
Riva services and model were deployed on rather limited computational powers:
```console
nvidia-smi
```
![image](https://user-images.githubusercontent.com/122811954/227789877-f7fc96ab-06ed-42d5-b770-40c48ef44435.png)
```console
lscpu
```
![image](https://user-images.githubusercontent.com/122811954/227789969-9ee35c60-a09b-4c34-b719-1d867ca9f93e.png)
```console
lscpu | egrep 'Model name|Socket|Thread|NUMA|CPU\(s\)'
```
![image](https://user-images.githubusercontent.com/122811954/227790076-fbaeb224-2d08-4154-b3c0-25a8c8e4f609.png)

While available GPU memory is 4095 MiB = 4294.967 MB; the chosen model, CTC-Conformer with offline recognition, requires 3100 MB.
# I: Installations 

## obtain ngc
```console
wget --content-disposition https://ngc.nvidia.com/downloads/ngccli_linux.zip && unzip ngccli_linux.zip && chmod u+x ngc-cli/ngc
find ngc-cli/ -type f -exec md5sum {} + | LC_ALL=C sort | md5sum -c ngc-cli.md5
echo "export PATH=\"\$PATH:$(pwd)/ngc-cli\"" >> ~/.bash_profile && source ~/.bash_profile
ngc config set
```

## obtain riva
```console
ngc registry resource download-version nvidia/riva/riva_quickstart:2.9.0
export NGC_API_KEY=<API_KEY>
```
### go to: in config.sh
[a full version may be seen HERE](config.sh)
#### choose models
```bash
service_enabled_asr=true
service_enabled_nlp=false
service_enabled_tts=false
service_enabled_nmt=false
```
#### config model
```bash
language_code=("en-US")
asr_acoustic_model=("conformer")
```
#### offline w/ CPU decoder
```bash
"${riva_ngc_org}/${riva_ngc_team}/rmir_asr_${asr_acoustic_model}_${modified_lang_code}_ofl${decoder}:${riva_ngc_model_version}"
```
To turn off Punctuation Checking using BERT (due to our computational resources) there is nedd to comment the following string in list of punctuation checkers for asr:

```bash
models_asr+=(
            #"${riva_ngc_org}/${riva_ngc_team}/models_nlp_punctuation_bert_base_${modified_lang_code}:${riva_ngc_model_version}-${riva_target_gpu_family}-${riva_tegra_platform}"
        )
```
This step is taken due to the limitation in GPU, which is not enough for deployment of BERT-based model.

It is turned on by default for all language-codes of ASR models.

(Also comment blocks of code with nlp/tts/mnt/punctuation models)

# II: Initializations
```console
cd riva_quickstart_v2.9.0
```
*new screen*
```console
sudo screen -S dockerd
sudo dockerd
```
*<ctrl+a+d>*

*new screen*
```console
sudo screen -S riva_ini_download
sudo bash riva_init.sh
```
*<ctrl+a+d>*

Check docker container status - e.g.:

```console
docker container ls
```
CONTAINER ID | IMAGE | COMMAND | CREATED | STATUS | PORTS | NAMES
|----------------------|----------------------|----------------------|----------------------|----------------------|--------------------------------------------|----------------------|
7f3f609403d7 | nvcr.io/nvidia/riva/riva-speech:2.9.0 | "/opt/nvidia/nvidia_…" | 57 seconds ago | Up 51 seconds | 0.0.0.0:50051->50051/tcp, :::50051->50051/tcp, 0.0.0.0:49155->8000/tcp, :::49155->8000/tcp, 0.0.0.0:49154->8001/tcp, :::49154->8001/tcp, 0.0.0.0:49153->8002/tcp, :::49153->8002/tcp | riva-speech


if eroors took place, it is recommended to run "sudo bash riva_clean.sh" and repeat init; though you may preserve containers by answering "n" to the first two prompts
```console
sudo bash riva_start.sh
```
# III: Starting client
```console
sudo bash riva_start_client.sh
```
# IV: Test sample
*inside container*
```console
cd wav
riva_asr_client --model_name conformer-en-US-asr-offline --audio_file=/opt/riva/wav/en-US_sample.wav --output_filename=./en-US_sample.txt
cat ./en-US_sample.txt
```

# V: Research sample format & FFMPEG ToolKit
In this part, we shall be precise any formats, codecs and etc.
We will use FFMPEG software for further work:
```console
apt-get update
apt-get install ffmpeg
ffmpeg --help
```

```console
ffmpeg -i /opt/riva/wav/en-US_sample.wav -f ffmetadata audio_spec.txt
```
![ffmpeg_1](https://user-images.githubusercontent.com/122811954/226694532-2afca5f9-25aa-458b-907f-05853860b7ce.png)
```console
cat audio_spec.txt
```
![ffmpeg_2](https://user-images.githubusercontent.com/122811954/226694732-937245a0-0fbf-48ef-9d6a-01a052462699.png)

To sum up, these parameters are our restrictions (Table below contains values and description for each parameter to ease further understanding of convertation and the process config):

| Property        | Value                       | Description                                                                                                                                                                                                                              |
|-----------------|-----------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Encoding format | pcm_s16le                   | Audio is recoreded and digitalized via *pulse-code modulation* using *signed int_16* values with *little endians (...le)* - byte word-writing order in which the "little end" (least significant value in the sequence) is stored first. |
| Frequency, Hz   | 16000                       | Sampling frequency of this file.                                                                                                                                                                                                         |
| Quality, Kbps   | 256                         | A second of audio is stored within 256 kilobits of data.                                                                                                                                                                                 |
| Encoder         | Lavf58.29.100/Lavf58.45.100 | Algorithm for audio encoding from *libav* - the ffmpeg project's fork.                                                                                                                                                                   |
| Audio channel   | Mono                        | Audio file includes only *one* audio track.                                                                                                                                                                                              |
