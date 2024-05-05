Tool to automatically convert your voice memos to text and analyze it with ChatGPT.
It consists of N8N workflows and uses Google Drive to sync and store all processed data.

![изображение](https://github.com/MrJimm/MemoAIzer/assets/5428408/9ae679ae-8d76-48ab-85cb-b147f8204217)

# TL;DR
1. Install n8n ([docker](https://docs.n8n.io/hosting/installation/docker/)).
2. Import voice_memo_transcribe.json workflow from json file.
3. Set up credentials for Google Drive and OpenAI nodes (see details at "Docs" tab of a node).
4. Create a root folder on your Google Drive, where all data will be stored
5. Link this root folder in "Root folder" node (dropdown near "from list" field)

      <img src="https://github.com/MrJimm/MemoAIzer/assets/5428408/8ff970a9-40fd-46af-9b2c-ec6d52f5e5d0" width="200">


7. Run the voice_memo_transcribe flow - it will automatically create all the folders hierarchy inside the root folder on your Google Drive.
8. Import, set up and run text analysis workflow (see section 2) 
9. Place some voice data into [root]/voice/raw/general and wait for results

NOTE: Since the memo transcription flow is triggered by a timer hourly, you can either run the flow manually with the 'Test workflow' button in the N8N UI to get immediate results, or adjust the timer trigger node (the first one) and restart the flow.

  <img src="https://github.com/MrJimm/MemoAIzer/assets/5428408/7433314f-af39-4c43-98f3-4d5add0ba8b2" width="600">


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

### 0.4 Google Drive folders hierarchy ()
After you run the voice transcription flow for the first time, it will create the 'voice' folder hierarchy within the selected root folder. It will also create a 'text/raw' folder, where all transcriptions will be stored.

```
|── root
  |── voice
      |── raw
            |── general
      |── processed
            |── general_processed
      |── error
            |── general_error
  |── text
      |── raw
      |── processed
            |── base
```

#### Voice
The "voice" folder is where all my input voice memo files are stored. Subfolders in the "raw" subfolder are where you put all your original voice memo files.
**Initially, there's only the "general" subfolder – start by placing your voice memo files here.**

Subfolders in the "processed" folder are where the flow will copy voice memo files after successful transcription. This allows the flow to detect new memos. I used the "copy" operation instead of "move" here because the tool I use to automatically upload voice memos from my smartwatch checks the files list in the original directory. Also, it's safer to use non-destructive operations. :)

Subfolders in the "error" folder are where voice memos will be copied if the Whisper node returns an error during transcription (this usually happens if a voice memo file is too large or your quota has expired). This error won't stop the flow. If you want these memos to be reprocessed, just remove the files from the corresponding "error" folder.

#### Text
"text" folders are for text memos. All successful transcriptions are automatically placed in "raw" folder. You also can place there your custom .txt file with text memo for further processing.
All text analysis results are stored in "processed" folder.


## Part 1: Transcribe voice memo to text

### 1.1 Activate N8N flow
Finally, activate your flow. You can then go to the "Executions" or "All executions" tab to monitor automation activities.
The transcription flow is triggered by the timer node. The initial period is set to one hour, which you can adjust in the "Schedule Trigger" node (the first one in the flow).

Memoaizer works with voice memos stored in Google Drive, in *devices* subfolders within root/voice/raw. To set up automatic upload from your phone/watch see section "How to upload to Drive" below. Also see troubleshooting section if you see no execution is fired while you have new files in *device* folders. You can see executions in the designated tab in N8N interface.

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
1. If you have unprocessed memo files in your upload folder, but N8N don't fire execution - stop it and try to run with a "Test workflow" button in the UI. This way you can see the instant flow execution in realtime. Also try to adjust the "Schedule Trigger" timer period value.
2. Though voice transcription flow shuffles new unprocessed files, sometimes flow can stuck on "bad" file. For this I occasionaly check execution status and if I notice such problematic file, I manually move it to specific "*_error" folders, that I created in my "voice" folder.
3. If your text memo processing flow does nothing after you started it, try to delete and then add manually any text file in "processed" folder. This should pull the right trigger in the flow. 
