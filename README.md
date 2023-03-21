# I: installations 

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
(Also comment blocks of code with nlp/tts/mnt/punctuation models)

# II: initializations
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
# III: starting client
```console
sudo bash riva_start_client.sh
```
# IV: test sample
*inside container*
```console
cd wav
riva_asr_client --audio_file=/opt/riva/wav/en-US_sample.wav --output_filename=./en-US_sample.txt
cat ./en-US_sample.txt
```

# V: work with samples
In this part, we shall be precise any formats, codecs and etc.
We will use FFMPEG software for further work:
```console
apt-get update
apt-get install ffmpeg
```

```console
ffmpeg -i /opt/riva/wav/en-US_sample.wav -f audio_spec.txt
```
