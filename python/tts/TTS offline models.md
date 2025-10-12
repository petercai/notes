
# models

If you're manually downloading XTTS v2 files (like `speakers_xtts.pth`) on another computer and transferring them to your MacBook, here's where to place them:

### 📁 Where to Put XTTS v2 Files on Your MacBook

XTTS v2 typically expects model files to be in a specific directory structure. If you're using the [**`coqui-ai/xtts-v2`**](https://huggingface.co/coqui/XTTS-v2) implementation, here's what to do:

1. **Create a Local Model Directory**: You can place the files in a folder like: ~/.cache/tts/coqui/XTTS-v2/ or ./models/xtts-v2/
    
2. **Required Files**: Make sure the following files are in the same directory:
    
    - `model.pth`
    - `config.json`
    - `speakers_xtts.pth`
3. **Set Environment Variables or Modify Code**: If the XTTS code tries to download from Hugging Face, you can:
    
    - Modify the script to load from your local path.
    - Or set an environment variable like:```Shell
        export TTS_MODELS_PATH=/path/to/models
        ```
        
4. **Using XTTS with Local Files**: When initializing the model in code, specify the local path:
    
    ```Python
    from TTS.api import TTS
    tts = TTS(model_path="/path/to/models/xtts-v2")
    ```
    

---

Would you like help writing a script to load XTTS v2 from local files, or setting up the environment on your MacBook?