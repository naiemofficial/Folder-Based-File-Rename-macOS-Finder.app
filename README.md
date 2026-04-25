# Folder-Based-File-Rename-macOS-Finder.app

<p>
A macOS Automator <b>Quick Action</b> that renames files using their parent folder name.  
Supports both <b>Prefix</b> and <b>Suffix</b> modes, with optional <b>recursive processing</b>.
</p>

<p align="center"><img width="1728" height="1084" alt="Screenshot 2026-04-26 at 12 59 45 AM" src="https://github.com/user-attachments/assets/933d88db-b378-409a-b321-1383f873142e" /></p>


<hr>

<h2>📦 About This Workflow</h2>

<p>This workflow integrates directly into Finder (right-click → Quick Actions) and allows you to:</p>

<ul>
  <li>Rename files based on their <b>containing folder name</b></li>
  <li>Choose naming format:
    <ul>
      <li><b>Prefix:</b> <code>Foldername - Filename.jpg</code></li>
      <li><b>Suffix:</b> <code>Filename - Foldername.jpg</code></li>
    </ul>
  </li>
  <li>Process:
    <ul>
      <li>Single folder</li>
      <li>Multiple folders</li>
      <li>Subfolders (recursive)</li>
    </ul>
  </li>
</ul>

<p><b>Use cases:</b> organizing photos, exported files, backups, and structured datasets, finding files by memorizing the folder name.</p>

<hr>

<h2>⚙️ Setup Steps</h2>

Download Import the workflow file from here <a href="/Folder-Based File Rename.workflow.zip">Folder-Based File Rename.workflow.zip</a>, follow the Workflow Configuration

<h3>🧱 Workflow Configuration</h3>

<ul>
  <li>Workflow type: <code>Quick Action</code></li>
  <li>Workflow receives current: <code>folders</code></li>
  <li>In: <code>Finder.app</code></li>
  <li>Input: <code>entire selection</code></li>
</ul>

<hr>

<h3>🧠 Step 1: Add Run AppleScript</h3>
Paste the below code

```bash
on run {input, parameters}
	-- Menu 1: Suffix/Prefix (ESC to abort)
	set theChoice to choose from list {"Suffix", "Prefix"} with prompt "Add folder name as:" default items {"Suffix"} OK button name "Next"
	if theChoice is false then error number -128
	
	-- Menu 2: Recursive (ESC to abort)
	try
		set isRecursive to button returned of (display dialog "Process all subfolders?" buttons {"Cancel", "No", "Yes"} default button "Yes" cancel button "Cancel")
	on error
		error number -128
	end try
	
	return {theChoice as text, isRecursive} & input
end run

```

<p>This step handles user interaction:</p>

<ul>
  <li>Prompt to choose:
    <ul>
      <li><code>Suffix</code></li>
      <li><code>Prefix</code></li>
    </ul>
  </li>
  <li>Prompt:
    <ul>
      <li><code>Process all subfolders?</code></li>
      <li>Options: <code>Yes</code> / <code>No</code></li>
    </ul>
  </li>
</ul>

<p><b>Behavior:</b></p>
<ul>
  <li>Pressing ESC cancels safely</li>
  <li>Returns selected mode + recursion preference + folders</li>
</ul>

<hr>

<h3>🖥️ Step 2: Run Shell Script</h3>
<ul>
  <li>Shell: <code>/bin/zsh</code></li>
  <li>Pass input: <code>as arguments</code></li>
</ul>


Paste the below code

```bash
choice="$1"
recursive="$2"
shift 2

for folder in "$@"; do
    if [[ "$recursive" == "Yes" ]]; then
        find_cmd="find \"$folder\" -type f -not -name \".*\""
    else
        find_cmd="find \"$folder\" -maxdepth 1 -type f -not -name \".*\""
    fi

    eval $find_cmd | while read -r f; do
        dir=$(dirname "$f")
        base=$(basename "$f")
        parent=$(basename "$dir")
        
        if [[ "$base" == *.* ]]; then
            ext=".${base##*.}"
            name="${base%.*}"
        else
            ext=""
            name="$base"
        fi

        if [[ "$choice" == "Prefix" ]]; then
            [[ "$name" != "$parent - "* ]] && mv -n "$f" "$dir/${parent} - ${name}${ext}"
        else
            [[ "$name" != *" - $parent" ]] && mv -n "$f" "$dir/${name} - ${parent}${ext}"
        fi
    done
done
```

<p><b>What it does:</b></p>

<ul>
  <li>Loops through selected folders</li>
  <li>Uses <code>find</code>:
    <ul>
      <li>Recursive → all subfolders</li>
      <li>Non-recursive → current folder only</li>
    </ul>
  </li>
  <li>Extracts:
    <ul>
      <li>File name</li>
      <li>Extension</li>
      <li>Parent folder name</li>
    </ul>
  </li>
  <li>Renames files:
    <ul>
      <li><b>Prefix:</b> <code>Foldername - Filename.ext</code></li>
      <li><b>Suffix:</b> <code>Filename - Foldername.ext</code></li>
    </ul>
  </li>
</ul>

<hr>

<h2>⚠️ Important: Dry Run Recommended</h2>

<p>Always test before applying to important files.</p>

<h3>🧪 Dry Run Steps</h3>

<ol>
  <li>Create a test folder:
    <pre>TestFolder/
  image1.jpg
  doc1.pdf</pre>
  </li>

  <li>Duplicate it:
    <pre>TestFolder_Copy/</pre>
  </li>

  <li>Run the workflow on <code>TestFolder_Copy</code></li>

  <li>Test combinations:
    <ul>
      <li><code>Prefix</code></li>
      <li><code>Suffix</code></li>
      <li>Recursive: <code>No</code> first</li>
    </ul>
  </li>

  <li>Verify:
    <ul>
      <li>Filenames are correct</li>
      <li>Extensions are preserved</li>
      <li>No unwanted overwrites</li>
    </ul>
  </li>
</ol>

<hr>

<h2>💡 Tips</h2>

<ul>
  <li>Start small before running on large folders</li>
  <li>Avoid running on original data without backup</li>
  <li>Best used on structured folder collections</li>
</ul>
