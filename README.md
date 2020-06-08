## Demo0 (speech_to_text, language_translator and text_to_speech)


<!-- curl --X GET -u "62a52d94-88f0-4ceb-8443-948644ec1fd7":"JpTYyY7A4Ppa" "https://stream-fra.watsonplatform.net/speech-to-text/api/v1/models" | jq -r '.[] | .[].language'


curl --cookie cookies.txt --cookie-jar cookies.txt -X POST -u ${CRED} ${URL}

curl -X POST -u ${CRED} --header "Content-Type: audio/mp3" --header "Transfer-Encoding: chunked" --data-binary @0.mp3 ${URL}'/recognize?model=fr-FR_BroadbandModel' | tee s2t0.resp0.json

curl -X POST -u ${CRED} --header "Content-Type: audio/mp3" --header "Transfer-Encoding: chunked" --data-binary @sound0-0.mp3  ${URL}'/recognize?model=fr-FR_BroadbandModel&speaker_labels=true&timestamps=true&word_alternatives_threshold=0.9&keywords=%22bonjour%22%2C%22retard%22&keywords_threshold=0.5' | tee sound0-0.resp1.json

	youtube-dl https://youtu.be/7CnIP3sblrE

# convert from mp3 (narrowband 8000Hz - amr) to mp3 (broadband)
avconv -i 2.mp3 -ar 22050 5.mp3
-->

<br>		

> To keep your environment clean we will complete this tutorial in a container

<br>		

#### Create demo0 container and connect to it
	docker search --stars=10 debian
	docker pull debian
	docker create --tty --interactive --name="demo0" debian:latest
	docker start demo0
	docker attach demo0
	apt-get update
	apt-get -y install locales locales-all
	sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
	locale-gen
	export LANG="en_US.UTF-8"  
	export LANGUAGE="en_US:en"  
	export LC_ALL="en_US.UTF-8"
	
#### Install ibmcloud tool, jq, curl, moreutils...
	apt-get install -y curl jq moreutils
	
	curl -fsSL https://clis.ng.bluemix.net/install/linux | sh	
	
#### Add some aliases	
```
cat >> ~/.bashrc << EOF
export ORG=teatcher0@bpshparis.com
export SPACE=dev
export BM_USER=teatcher0@bpshparis.com
export US_REGION=https://api.ng.bluemix.net
export GB_REGION=https://api.eu-gb.bluemix.net
export DE_REGION=https://api.eu-de.bluemix.net
alias iclus='/usr/local/bin/ibmcloud login -a ${US_REGION} -u ${BM_USER} --skip-ssl-validation -s ${SPACE} -o ${ORG}'
alias iclgb='/usr/local/bin/ibmcloud login -a ${GB_REGION} -u ${BM_USER} --skip-ssl-validation -s ${SPACE} -o ${ORG}'
alias iclde='/usr/local/bin/ibmcloud login -a ${DE_REGION} -u ${BM_USER} --skip-ssl-validation -s ${SPACE} -o ${ORG}'
alias ic='/usr/local/bin/ibmcloud'
export S2T_SVC=s2t0
export LT_SVC=lt0
export T2S_SVC=t2s0
export SVC_KEY=user0
alias l='ls -Alhtr'
EOF
```

#### Add aliases to your environment
	. ~/.bashrc

<br>

### Log to IBM Cloud in Germany
	ibmcloud login -a ${DE_REGION} -u ${BM_USER} --skip-ssl-validation -s ${SPACE} -o ${ORG}
	
or use the alias

	iclde

> If **login failed** then get a one time code here:
https://login.eu-de.bluemix.net/UAALoginServerWAR/passcode
and login with **--sso**

	ibmcloud login -a ${DE_REGION} -u ${BM_USER} --sso -s ${SPACE} -o ${ORG}

> Paste one time code when prompt
One Time Code (Get one at https://login.eu-de.bluemix.net/UAALoginServerWAR/passcode)>
and hit enter.

<br>

### Dump marketplace to get service name, plan and description
> It may take a minute to display

	ibmcloud service offerings | tee marketplace

<br>

### Start working with Text to Speech service

#### Get name and plan for Speech to Text service
	grep -i speech marketplace

#### Create Speech to Text service
	ibmcloud service create speech_to_text lite ${S2T_SVC}

#### Create service key (credential) for Speech to Text service
	ibmcloud service key-create ${S2T_SVC} ${SVC_KEY}

#### Build url to use Speech to Text service and store it in URL environment variable
	URL=$(ic service key-show ${S2T_SVC} ${SVC_KEY} | awk 'NR >= 4 {print}' | jq -r '.url + "/v1/recognize"')

#### Build credential string to  use Speech to Text service and store it in CRED environment variable
	CRED=$(ic service key-show ${S2T_SVC} user0 | awk 'NR >= 4 {print}' | jq -r '.username + ":" + .password')
	
or	

	CRED=$(ic service key-show ${S2T_SVC} user0 | awk 'NR >= 4 {print}' | jq -r '"apikey:" + .apikey')
	
#### Store a sound file path in SOUND environment variable

> To copy a sound file in demo0 container,
From a host shell execute:

	docker cp sound0.mp3 demo0:/root
	
> then 

	SOUND=$(readlink -f sound0.mp3)

#### Choose a language model among this list and append it to URL environment variable

 * ar-AR_BroadbandModel
 * en-GB_BroadbandModel
 * en-GB_NarrowbandModel
 * en-US_BroadbandModel
 * en-US_NarrowbandModel
 * es-ES_BroadbandModel
 * es-ES_NarrowbandModel
 * **fr-FR_BroadbandModel**
 * ja-JP_BroadbandModel
 * ja-JP_NarrowbandModel
 * ko-KR_BroadbandModel
 * ko-KR_NarrowbandModel
 * pt-BR_BroadbandModel
 * pt-BR_NarrowbandModel
 * zh-CN_BroadbandModel
 * zh-CN_NarrowbandModel

#### Store the chosen language model in MODEL environment variable
	MODEL=fr-FR_BroadbandModel

#### Send the request to Speech to Text service using credential and url from above and store the result in a file
	curl -X POST -u ${CRED} --header 'Content-Type: audio/mp3' --header 'Transfer-Encoding: chunked' --data-binary @${SOUND} ${URL}'?model='${MODEL} | tee ${S2T_SVC}.resp0.json

#### Store transcription in TRANSCRIPT environment variable
	TRANSCRIPT=$(jq -r '.results[].alternatives[].transcript' s2t0.resp0.json)

<br>

## Start working with Language Translator service

#### Find Language Translator service name and available plans in local file generated above
	grep -i translator marketplace

#### Create Language Translator service
	ibmcloud service create language_translator lite ${LT_SVC}

#### Create service key (credential) for Language Translator service
	ibmcloud service key-create ${LT_SVC} ${SVC_KEY}

#### Build url to use to use Language Translator service and store it in URL environment variable
	URL=$(ibmcloud service key-show ${LT_SVC} ${SVC_KEY} | awk 'NR >= 4 {print}' | jq -r '.url + "/v3/translate?version=2018-05-01"')

#### Build credential string to  use Language Translator service  and store it in CRED environment variable
	CRED=$(ibmcloud service key-show ${LT_SVC} ${SVC_KEY} | awk 'NR >= 4 {print}' | jq -r '("apikey:" + .apikey)')

#### Create a parameter file to request Language Translator service
```
cat > ${LT_SVC}.req0.parms.json << EOF
{
  "text": "",
  "source": "fr",
  "target": "en"
}
EOF
```

#### Update Language Translator service parameter file with value from TRANSCRIPT environment variable
	jq --arg NEWTEXT "${TRANSCRIPT}" '.text = $NEWTEXT' ${LT_SVC}.req0.parms.json | sponge ${LT_SVC}.req0.parms.json

> If needed remove line feed

	sed -i 's/\\n/ /g' ${LT_SVC}.req0.parms.json

#### Send the request to Language Translator service and store the result in a file
	curl -X POST -u ${CRED} -H 'Content-Type: application/json' -H 'Accept: application/json' -d @${LT_SVC}.req0.parms.json ${URL} | tee ${LT_SVC}.resp0.json

#### Store translation in TRANSLATION environment variable
	TRANSLATION=$(jq -r '.translations[].translation' ${LT_SVC}.resp0.json)

<br>

### Start working with Text to Speech service

#### Find Text to Speech service name and available plans in local file generated above
	grep -i Speech marketplace

#### Create Text to Speech service
	ibmcloud service create text_to_speech lite ${T2S_SVC}

#### Create service key (credential) for Text to Speech service
	ibmcloud service key-create ${T2S_SVC} ${SVC_KEY}

#### Build url to use to use Text to Speech service and store it in URL environment variable
	URL=$(ibmcloud service key-show ${T2S_SVC} ${SVC_KEY} | awk 'NR >= 4 {print}' | jq -r '.url + "/v1/synthesize"')

#### Build credential string to  use Text to Speech service and store it in CRED environment variable
	CRED=$(ibmcloud service key-show ${T2S_SVC} ${SVC_KEY} | awk 'NR >= 4 {print}' | jq -r '.username + ":" + .password')

#### Choose a voice among this list and append it to URL environment variable
 * de-DE_BirgitVoice
 * de-DE_DieterVoice
 * en-GB_KateVoice
 * en-US_AllisonVoice
 * en-US_LisaVoice
 * **en-US_MichaelVoice**
 * es-ES_LauraVoice
 * es-ES_EnriqueVoice
 * es-LA_SofiaVoice
 * es-US_SofiaVoice
 * fr-FR_ReneeVoice
 * it-IT_FrancescaVoice
 * ja-JP_EmiVoice
 * pt-BR_IsabelaVoice
 
#### Store the chosen voice in VOICE environment variable
	VOICE=en-US_MichaelVoice

#### Create a parameter file to request Text to Speech service
```
cat > ${T2S_SVC}.req0.parms.json << EOF
{
  "text": ""
}
EOF
```

#### Update Text to Speech service parameter file with value from TRANSLATION environment variable
	jq --arg NEWTEXT "${TRANSLATION}" '.text = $NEWTEXT' ${T2S_SVC}.req0.parms.json | sponge ${T2S_SVC}.req0.parms.json

#### Send the request to  Text to Speech service and store the result in a file
	curl -u ${CRED} -X POST -H 'Content-Type: application/json' -H 'Accept: audio/mp3' --output ${T2S_SVC}.resp0.mp3 -d @${T2S_SVC}.req0.parms.json ${URL}'?voice='${VOICE}


> To copy the result sound file out of the container,
from a host shell execute:

	docker cp demo0:/root/t2s0.resp0.mp3 .

<br>

### Clean your room

	for svc in ${T2S_SVC} ${LT_SVC} ${S2T_SVC}; do ibmcloud service key-delete -f $svc ${SVC_KEY}; ibmcloud service delete -f $svc; done

<br>

### Bring your work with you

#### Stop demo0 container 
> From your container

	exit
	
> From you host

	docker stop demo0
	
#### Create a image from your container
	docker commit demo0 bpshparis/demo0:latest
	
#### Save the image in a file
	docker save bpshparis/demo0 | gzip -c > bpshparis.demo0.tar.gz

> To restore your image in another environment execute:

	docker load < bpshparis.demo0.tar.gz

Then repeat steps from [Create demo0 container and connect to it](#create-demo0-container-and-connect-to-it) to create a container.




