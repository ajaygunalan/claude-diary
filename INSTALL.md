# Installation Guide

This guide will help you install and set up the Claude Code Memory System plugin.

## Prerequisites

- Claude Code installed and working
- Access to your `~/.claude` directory
- Basic familiarity with command line

## Installation Methods

### Method 1: Local Marketplace (Recommended)

This method allows you to manage the plugin like any other Claude Code plugin.

1. **Clone or download this repository**:
   ```bash
   cd ~/Desktop/Code  # or wherever you keep your code
   git clone https://github.com/ajaygunalan/claude-diary cc-memory
   # OR download and extract the ZIP file
   ```

2. **Create a local marketplace directory** (if you don't have one):
   ```bash
   mkdir -p ~/.claude/marketplaces/local
   ```

3. **Create or edit your marketplace.json file**:
   ```bash
   nano ~/.claude/marketplaces/local/marketplace.json
   ```

4. **Add the plugin to your marketplace.json**:
   ```json
   {
     "name": "local-plugins",
     "owner": {
       "name": "Local"
     },
     "plugins": [
       {
         "name": "claude-memory",
         "source": "/Users/YOUR_USERNAME/Desktop/Code/cc-memory",
         "description": "Long-term memory system for Claude Code"
       }
     ]
   }
   ```

   **Important**: Replace `/Users/YOUR_USERNAME/Desktop/Code/cc-memory` with the actual path where you cloned/downloaded the plugin.

5. **Install the plugin in Claude Code**:
   - Open Claude Code
   - Run the plugin install command (check Claude Code docs for current command)
   - Select "local-plugins" marketplace
   - Select "claude-memory" plugin
   - Confirm installation

6. **Verify installation**:
   ```bash
   # In Claude Code, try:
   /diary
   /reflect
   ```

### Method 2: Manual Skill Installation

This method copies just the skills without full plugin structure.

1. **Copy skills to your global skills directory**:
   ```bash
   cp -r skills/* ~/.claude/skills/
   ```

2. **Verify installation**:
   ```bash
   # In Claude Code, try:
   /diary
   /reflect
   ```

**Note**: With this method, you won't get automatic updates if the plugin is updated.

## Post-Installation Setup

### 1. Create Diary Directories

The plugin will create these automatically on first use, but you can create them manually if you prefer. Diary data is stored per-project under `~/.claude/diary/<project>/` (where `<project>` is the git repo name, consistent across worktrees):

```bash
PROJECT_NAME=$(git remote get-url origin 2>/dev/null | sed 's|.*/||; s|\.git$||' || basename "$(pwd)")
mkdir -p ~/.claude/diary/${PROJECT_NAME}
mkdir -p ~/.claude/diary/${PROJECT_NAME}/reflections
```

### 2. Verify Permissions

Make sure you have write permissions:

```bash
ls -la ~/.claude/diary/
```

You should see a directory for each project you've used the plugin with.

### 3. Test the Plugin

1. **Create a test diary entry**:
   - Do some work in Claude Code (any session)
   - Run `/diary`
   - Check that a file was created: `ls -la ~/.claude/diary/<project>/`

2. **Test reflection** (after you have 2-3 diary entries):
   - Run `/reflect`
   - Verify a reflection file was created: `ls -la ~/.claude/diary/<project>/reflections/`

### 4. Set Up PreCompact Hook (Optional - For Automatic Diary Generation)

The PreCompact hook automatically generates diary entries before Claude Code compacts long conversations (200+ messages).

**Step 1: Install the hook script**

```bash
mkdir -p ~/.claude/hooks
cp hooks/pre-compact.sh ~/.claude/hooks/pre-compact.sh
chmod +x ~/.claude/hooks/pre-compact.sh
```

**Step 2: Configure the hook in settings**

Add the following to `~/.claude/settings.json` (global) or `.claude/settings.local.json` (project-specific):

```json
{
  "hooks": {
    "PreCompact": [
      {
        "matcher": "auto",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/pre-compact.sh"
          }
        ]
      }
    ]
  }
}
```

**Alternative: Use the interactive command**

```bash
/hooks
```

Then select:
- Hook event: `PreCompact`
- Matcher: `auto` (for automatic compaction) or `manual` (for `/compact` command)
- Command: `bash ~/.claude/hooks/pre-compact.sh`

**Verify hook configuration**

Check that your settings file contains the hook configuration:

```bash
cat ~/.claude/settings.json | grep -A 10 "PreCompact"
```

## Troubleshooting

### "Command not found: /diary"

**Problem**: Claude Code doesn't recognize the skill.

**Solutions**:
1. Check that the skill file exists: `ls ~/.claude/skills/diary/SKILL.md`
2. Restart Claude Code
3. Try the full plugin installation method instead

### "Cannot find session transcript"

**Problem**: The `/diary` command can't locate your session files.

**Solutions**:
1. Make sure you're running `/diary` from within a Claude Code session (not a regular terminal)
2. Check that session files exist: `ls ~/.claude/projects/`
3. Try running `/diary` at the end of a session, not during

### "Permission denied" creating directories

**Problem**: Can't write to `~/.claude/diary/`

**Solutions**:
1. Check ownership: `ls -la ~/.claude/`
2. Fix permissions: `chmod 755 ~/.claude/diary/`
3. Manually create directories: `PROJECT_NAME=$(git remote get-url origin 2>/dev/null | sed 's|.*/||; s|\.git$||' || basename "$(pwd)") && mkdir -p ~/.claude/diary/${PROJECT_NAME}/reflections`

### "No diary entries found" when running /reflect

**Problem**: `/reflect` can't find any diary entries.

**Solutions**:
1. Run `/diary` first to create some entries
2. Check that diary files exist: `ls ~/.claude/diary/<project>/`
3. Verify file permissions: `ls -la ~/.claude/diary/<project>/`

### PreCompact hook not running

**Problem**: The hook script exists but diary entries aren't being automatically generated before compaction.

**Solutions**:
1. Check that the hook is configured in `settings.json`:
   ```bash
   cat ~/.claude/settings.json | grep -A 10 "PreCompact"
   ```
2. Verify the hook script is executable:
   ```bash
   ls -la ~/.claude/hooks/pre-compact.sh
   # Should show -rwxr-xr-x permissions
   ```
3. Make it executable if needed:
   ```bash
   chmod +x ~/.claude/hooks/pre-compact.sh
   ```
4. Check that you have the correct hook configuration (both the script file AND the settings.json configuration are required)
5. Restart Claude Code after adding hook configuration

## Updating the Plugin

### If installed via marketplace:
1. Pull the latest changes: `cd ~/Desktop/Code/cc-memory && git pull`
2. Restart Claude Code (changes should be picked up automatically)

### If installed manually:
1. Pull the latest changes: `cd ~/Desktop/Code/cc-memory && git pull`
2. Copy updated skills:
   ```bash
   cp -r skills/* ~/.claude/skills/
   ```
3. Restart Claude Code

## Uninstalling

### If installed via marketplace:
- Use Claude Code's plugin uninstall command

### If installed manually:
```bash
rm -r ~/.claude/skills/diary
rm -r ~/.claude/skills/reflect
```

**Note**: This does not delete your diary entries or reflections. To remove those:
```bash
rm -rf ~/.claude/diary/  # removes all project diary data
```

## Next Steps

Once installed:
1. Read the [README.md](README.md) for usage instructions
2. Check [examples/sample-diary-entry.md](examples/sample-diary-entry.md) to see what diary entries look like
3. Check [examples/sample-reflection.md](examples/sample-reflection.md) to see what reflections look like
4. Start using `/diary` after your Claude Code sessions!

## Getting Help

- Check the [README.md](README.md) for usage guidelines
- Check Claude Code documentation for general plugin issues
- Create an issue in the repository if you find bugs
