# MemoAIzer
Tool to automatically convert your voice memos to text and analyze it with ChatGPT.
It consists of N8N workflows and uses Google Drive to sync and store all processed data.

![изображение](https://github.com/MrJimm/MemolAIzer/assets/5428408/63926911-90f7-4bc6-917e-901c57615ad9)

# TL;DR
1. Install n8n ([docker](https://docs.n8n.io/hosting/installation/docker/)).
2. Import workflows from json files.
3. Set up credentials for Google Drive and OpenAI.
4. Create necessary folders on Google Drive (see details in section 0.4 below).
5. Reconnect folders in Google Drive nodes to the folders created in step 4 (see sections 0.4, 0.5, 2.1).
6. Start flows.
7. Place some data in input folders and observe.

# Detailed instruction

## Part 0: Setup prerequisites
### 0.1 Install N8N
Install N8N from the official website, or get official docker image from hub (seem easier way).

Set N8N_PAYLOAD_SIZE_MAX env variable to some high value (123456 will be enough), so you can move large files in your N8N flow over the internet.

If you use docker image - set this env variable for the container before you run it. Also don't forget to forward the port (default is 5678).

### 0.2 Start N8N and import workflow
Start N8N service/container and open web interface (the default is http://localhost:5678).

At first start it will require you to pass the simple sign up procedure.

Then create a new workflow, press three dots button at right upper corner, select "Import from File" and select file of Voice memo transcribe flow (voice_memo_transcribe.json).
Save the flow (also you can change its name, if you click on its name "My workflow" in the upper panel).

### 0.3 Create OpenAI and Google Drive credentials
Open any Google Drive node (i.e. "New in general folder") and follow instructions on Docs tab to create credentials, that you can use to connect to your Google Drive account.

![изображение](https://github.com/MrJimm/MemolAIzer/assets/5428408/3944f783-c4ae-4e89-9564-38a8bca25b0e)

After you created Drive credentials, open each Google Drive node and choose it. Do the same for Whisper transcription node to create OpenAI API credentials.

### 0.4 Create Google Drive folders hierarchy 
Go to your Google Drive and create "memo" folder somewhere. I have the following folder structure. 

```
|── memo
  |── voice
      |── watch
      |── general
      |── ipad
      |── iphone
      |── ...
      |── watch_processed
      |── general_processed
      |── ipad_processed
      |── iphone_processed
      |── ...
  |── text
      |── raw
      |── processed
        |── base
```

#### Voice
"voice" folder is where all my input voice memo files are stored. I devided to have separate folder for each of my device, to be sure that memos with the same names will not overwrite each other. But for start you can leave only "general" folder. Don't forget to remove triggers for other folders in voice memo transcribe flow.
If you need to add new input folder for voice memos - create flow entrance with trigger, similar to others:

![изображение](https://github.com/MrJimm/MemolAIzer/assets/5428408/d2fc5ae3-3246-4d31-9aab-934a93d9e76d)

!!!

**BEWARE! THIS FLOW MAKE CHANGES ON YOUR GOOGLE DRIVE, MODIFIES FILES IN SPECIFIC FOLDERS! THOUGH INTENT IS TO CHANGE ONLY FILES AND FOLDERS CREATED ONLY BY THIS TOOL AND FOR THIS TOOL, SOME OF THEM ARE SOUGHT GLOBALLY BY NAME. MAKE SURE THAT YOU HAVE UNIQUE _processed FOLDERS!**

**BE CAREFUL TO USE THIS TOOL AND CHECK TWICE ALL GOOGLE DRIVE WRITE NODES YOURSELF IF YOU HAVE ANY IMPORTANT DATA IN YOUR GOOGLE DRIVE, SINCE THIS SOFTWARE IS UNDER DEVELOPMENT AND HAVEN'T BEEN FULLY TESTED! IT CAN BE GOOD IDEA TO USE A SEPARATE DRIVE ACCOUNT IF YOU HAVE ONE**

!!!

"*_processed" folders are technical, but vital for tool to work. When the flow successfully transcribe voice memo from the "*device*" folder, it will copy its file to the correspondent "_processed" folder. And when the flow execution is triggered, it will process all files from all "*device*" folders, that are not found in "_processed" folders. This is done to prevent skipping some files if some "new file" trigger events are lost from Google Drive. You can use this to mark memo file for reprocessing - just delete correspondant file from "_processed" folder.

#### Text
"text" folders are for text memos. All successful transcriptions are automatically placed in "raw" folder. You also can place there your custom .txt file with text memo for further processing.
All text analysis results are stored in "processed" folder. This topic is WIP for now. 

### 0.5 Set up Google Drive folders in flow
After you created folders, double click on every Google Drive trigger node and reselect your *device* folders that you have created. Click on a dropdown list to the right of "from list" and find your folder by name

![изображение](https://github.com/MrJimm/MemolAIzer/assets/5428408/57e1f926-1cf9-415f-bf3a-897893d48605)

Go to the "Search if the text memo already exists" node and reselect your "raw" text folder.

Go to the Google Drive node after the "MAKE JSON FILES" and reselect your "raw" text folder. 

## Part 1: Transcribe voice memo to text

### 1.1 Activate N8N flow
Finally activate your flow. You can go then to "Executions" or "All executions" tab to monitor automation activities.

Memoaizer works with voice memos stored in Google Drive. The flow is triggered when any new file is uploaded to any of the voice/[device] folders (general, watch, etc). To set up automatic upload from your phone/watch see section "How to upload to Drive" below. Also see troubleshooting section if you see no execution is fired while you have new files in *device* folders. You can see executions in the designated tab in N8N interface.

## Part 2: Analyze text transcriptions with ChatGPT API 
Text Memo analysis flow from a text_memo_analysis.json will analyze your text transcriptions with ChatGPT API. The base processing for each memo are: 

1. XML with two titles and keywords, 
2. Original text, formatted and cleaned
3. Summary of the note in the original language
4. Summary of the note in English
5. Copy of the original text of the transcription

### 2.1 Load text analysis flow from the file
1. Load text memo analysis flow from the text_memo_analysis.json.
2. For evey Google Drive and OpenAI API block set-up credentials as you did above.

![изображение](https://github.com/MrJimm/MemoAIzer/assets/5428408/928452cf-32fc-4f77-9756-7071024ec45c)

(red exclamation mark signs near Google Drive and OpenAI blocks highlights where you should set up credentials)

3. On your Google Drive in your text/processed/ folder create "base" folder (see folders hierarchy in step 0.4)
4. Reselect your "base" folder in both "Get all items in processed base folder" and "Create output folder" nodes
   
![изображение](https://github.com/MrJimm/MemoAIzer/assets/5428408/cd5f5b98-b223-47fa-a3ef-e9f3b138f472)


6. Reselect "raw" folder in "Get all items in input folder" node
   
![изображение](https://github.com/MrJimm/MemoAIzer/assets/5428408/c949cf55-7436-4a5b-b75c-e6731a02a02f)



7. Launch the flow

### 2.2 Managing text processing prompts
You can find prompts for text processing inside OpenAI blocks (i.e. "Get keywords"). It is plain text, and I suggest to read it to understand how exactly it asks ChatGPT to process your text memos. You can change this prompts and add new branches with custom processing prompts. Though I plan to introduce a more convenient way to add custom text processing flows in further updates.

## How to upload to Drive 
### Manually
It is what it is:) Just place voice memo file it in the specific (i.e. "general") folder.
Also you can place text memos (in .txt format) straight to the text/raw folder, so they will be processed as regular transcriptions. But be aware of the size of context window of GPT model you use, so your custom text will fit it.

### Android
I use pro version of FolderSync app to automatially sync recordings files from some particular folder on a phone to "watch" folder on my Google Drive. This app allows to speify particular folder, where all recordings only from watch are stored, and "to" folder on Google Drive.  

### Watch: example for Galaxy Watch 3 and Samsung S23 Ultra:
For Galaxy Watch 3 on Android in the Wearable app go to Apps -> Voice Recorder -> Settings (gear icon) and toggle "Auto copy to phone". 
Then all recordings will sync in the folder /storage/emulated/0/Recordings/Sounds/Gear (at least, on my setup. Check similar path if memos will not appear there). Now you can use this path on your Phone to your watch recordings in sync app (FolderSync Pro in my case) to automatically sync memos to Drive ("watch" folder in my case).

## Troubleshooting
1. If you have unprocessed memo files in your upload folder, but N8N don't fire execution - try to stop flow, start it, and only then remove one of the new files from the folder on Drive and place it back. Seems, that sometimes triggers on new files works if these files were placed only after the N8N flow was started.
2. If you want to test the flow, but have any of your *device* folders empty, then you may encounter that hitting "test workflow" on voice memo transcription flow leads to error. Try to disconnect temporarily starting "trigger" brunches of empty folders. Or you can fetch test data if you open the node (by double clicking on it).
3. Though voice transcription flow shuffles new unprocessed files, sometimes flow can stuck on "bad" file. For this I occasionaly check execution status and if I notice such problematic file, I manually move it to specific "*_error" folders, that I created in my "voice" folder.
4. If your text memo processing flow does nothing after you started it, try to delete and then add manually any text file in "processed" folder. This should pull the right trigger in the flow. 
