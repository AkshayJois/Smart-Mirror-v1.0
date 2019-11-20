# Smart-Mirror-v1.0
Source code is taken from Michi Mich and makecodeplus repositories

RASPBERRY PI 

This example is about how to install google assistant and magic mirror on raspbian.

Dependency Versions

OS : 2019-04-08-raspbian-stretch-full.img
MagicMirror2 : 2.7.1
MMM-Hotword : 2.0.1
MMM-AssistantMk2 : 2.1.4

Install

STEP1. Raspbian installation
Download the latest image from the Raspbian website. (2019-04-08-raspbian)
Download Rufus to write images to SD card.
Run Rufus and select the downloaded image to burn the SD card.

STEP2. Insert SD card + LCD + Power connection
Insert SD card and keyboardㆍmouse dongle into raspberry pi.
Connect LCD to raspberry pi on HDMI.

STEP3. First Boot - PART1
After boot is done, connect to the Internet with WIFI
Install Remote Desktop Server
sudo apt-get install xrdp
Check the IP address with ipconfig

STEP3. First Boot - PART2
Remote access to raspberry pi. (ID: pi, password raspberry)
Update packages
sudo apt-get update

STEP4. Installing a Magic Mirror
Install the Magic Mirror using a script on the Internet
sudo apt-get install npm
sudo npm install -g npm@latest
bash -c "$(curl -sL https://raw.githubusercontent.com/MichMich/MagicMirror/master/installers/raspberry.sh)"

STEP5. Installing Magic Mirror Modules

Install dependencies
sudo apt-get install libmagic-dev libatlas-base-dev sox libsox-fmt-all mpg321 libasound2-dev

Install the MMM-Hotword
cd ~/MagicMirror/modules/
git clone https://github.com/eouia/MMM-Hotword.git
cd MMM-Hotword
chmod +x ./installer/install.sh
./installer/install.sh

Install MMM-AssistantMk2
cd ~/MagicMirror/modules/
git clone https://github.com/eouia/MMM-AssistantMk2
cd MMM-AssistantMk2
npm install
cd scripts
chmod +x *.sh
cd ..
npm install --save-dev electron-rebuild
./node_modules/.bin/electron-rebuild

STEP6. Configure Google Assistant Module



Open the Google Action Console and create a new project
https://console.actions.google.com

Open the Google Cloud Platform Console and select the generated project
https://console.cloud.google.com


Search for the Google Assistant API and click Enable.

Click CONFIGURE ... of Credentials and put the name and e-mail.

Generate Other credentials with the OAuth Client ID in Create Credentials

Download generated OAuth client ID in json format

Move the downloaded OAuth client ID to modules/MMM-AssistantMk2/credentials.json
mv ~/Download/cre.... credentials.json

Run auth_and_test.js to verify the generated client ID
node auth_and_test.js

Accept the client verification process and copy and enter your Google account key

Move the generated token.json
mv token.json ./profiles/default.json

STEP7. Edit Google assistant module config
Open the Magic Mirror configuration file with TextEditor and modify it with the contents of github

{
	module: "MMM-Hotword",
	position: "top_right",
	config: {
		chimeOnFinish: null,
		mic: {
			recordProgram: "arecord",
			device: "plughw:1"
		},
		models: [
			{
				hotwords    : "smart_mirror",
				file        : "smart_mirror.umdl",
				sensitivity : "0.5",
			},
		],
		commands: {
			"smart_mirror": {
				notificationExec: {
					notification: "ASSISTANT_ACTIVATE",
					payload: (detected, afterRecord) => {
						return {profile:"default"}
					}
				},
				restart:false,
				afterRecordLimit:0
			}
		}
	}
},
{
	module: "MMM-AssistantMk2",
	position: "top_right",
	config: {
		deviceLocation: {
			coordinates: {
				latitude: 37.5650168, // -90.0 - +90.0
				longitude: 126.8491231, // -180.0 - +180.0
			},
		},
		record: {
			recordProgram : "arecord",  
			device        : "plughw:1",
		},
		notifications: {
			ASSISTANT_ACTIVATED: "HOTWORD_PAUSE",
			ASSISTANT_DEACTIVATED: "HOTWORD_RESUME",
		},
		useWelcomeMessage: "brief today",
		profiles: {
			"default" : {
				lang: "en-US"
			}
		},
	}
},




Install USB Mic. and Speaker

https://github.com/makepluscode/rpi-tips/tree/master/001-bringup-audio-and-mic

vi ~/.asoundrc
pcm.!default{
  type asym
  playback.pcm{
    type hw
    card 0
  }
  capture.pcm{
    type plug
    slave.pcm "hw:1, 0"
  }
}

ctl.!default{
  type hw
  card 0
}
And, change audio output from hdmi to analog.

Test
Go to the location where the Magic Mirror is and start the application

cd ~/MagicMirror
npm start
