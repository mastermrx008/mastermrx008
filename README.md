## Hi there 👋

<!--
**mastermrx008/mastermrx008** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->#!/bin/bash

# Your API key
API_KEY="YOUR_API_KEY_HERE"

# Option to delete file after successful upload (true/false)
DELETE_AFTER_UPLOAD=false

# Function to upload a single file
upload_file() {
  FILE_PATH=$1
  FILE_NAME=$(basename $FILE_PATH)
  
  # Check file's MD5 to avoid duplicate uploads
  MD5=$(md5sum $FILE_PATH | awk '{ print $1 }')
  if grep -q "^$MD5" uploaded.txt; then
    echo "File $FILE_NAME already uploaded. Skipping."
    return
  fi

  # Send the file to Hydrax
  RESPONSE=$(curl -F "file=@$FILE_PATH" "up.hydrax.net/$API_KEY")
  SLUG=$(echo $RESPONSE | jq -r '.slug')

  # Check and save upload information
  if [ "$SLUG" != "null" ]; then
    echo "$MD5 | $FILE_NAME | $SLUG" >> uploaded.txt
    echo "Uploaded $FILE_NAME with slug $SLUG."
    # Delete the file if the option is set to true
    if [ "$DELETE_AFTER_UPLOAD" = true ]; then
      rm $FILE_PATH
      echo "File $FILE_NAME deleted after successful upload."
    fi
  else
    echo "Failed to upload $FILE_NAME."
  fi
}

# Check argument count
if [ "$#" -ne 1 ]; then
  echo "Usage: $0 /path/to/file/or/folder"
  exit 1
fi

# Check if the argument is a file or folder
if [ -f "$1" ]; then
  upload_file "$1"
elif [ -d "$1" ]; then
  for FILE_PATH in "$1"/*; do
    FILE_TYPE=$(file --mime-type -b "$FILE_PATH")
    if [[ "$FILE_TYPE" == video/* ]]; then
      upload_file "$FILE_PATH"
    fi
  done
else
  echo "Error: Path must be a valid file or folder."
  exit 1
fi
