```markdown
# YTMusic-Save-To-Library

**YTMusic-Save-To-Library** is a lightweight JavaScript tool that batch-saves unsaved tracks from your YouTube Music playlists into your library. The script scrolls through your playlist, reveals hidden menu options, and automatically clicks the “Save to library” option for each song.

## Features

- **Batch Save:** Automatically add unsaved songs to your library.
- **Automatic Scrolling:** Loads the full playlist by scrolling.
- **Menu Interaction:** Simulates clicks to reveal and use YouTube Music’s menu controls.
- **Easy Usage:** Run the script in your browser’s Developer Console—no installation needed.

## How It Works

1. **Scroll:** The script scrolls to the bottom of the page until all songs are loaded.
2. **Menu Trigger:** It queries for each song’s menu trigger element using the class  
   `.dropdown-trigger.style-scope.ytmusic-menu-renderer`
3. **Dropdown & Save:** For each song, the script clicks the trigger, waits for the dropdown to appear, and then looks for a menu item labeled exactly “Save to library” (or “Add to library”). If found, it clicks it.
4. **Cleanup:** It closes the dropdown and moves on to the next song.

## Instructions

1. Open your YouTube Music playlist (or "Liked Music") page. (It’s a good idea to scroll to the bottom manually first.)
2. Open your browser’s Developer Console (F12, Ctrl+Shift+J, or Cmd+Option+J).
3. Paste the entire script (provided below) into the console and press Enter.
4. Watch the console logs for progress. The script will process each song and update your library accordingly.

## Usage

```js
(async function () {
  // ─── Helper: Scroll to the bottom so that all songs load ─────────────────────────────
  async function scrollToBottom(delay = 1000) {
    console.log("Scrolling to load all songs...");
    let lastHeight = document.body.scrollHeight;
    while (true) {
      window.scrollTo(0, document.body.scrollHeight);
      await new Promise((r) => setTimeout(r, delay));
      let newHeight = document.body.scrollHeight;
      if (newHeight === lastHeight) break;
      lastHeight = newHeight;
    }
    console.log("Scrolling complete; all songs should now be loaded.");
  }

  // ─── Helper: Wait for the dropdown menu to appear ─────────────────────────────
  async function waitForDropdown(timeout = 5000) {
    return new Promise((resolve, reject) => {
      const start = Date.now();
      const interval = setInterval(() => {
        const dropdown = document.querySelector("ytmusic-menu-popup-renderer[slot='dropdown-content']");
        if (dropdown) {
          clearInterval(interval);
          resolve(dropdown);
        } else if (Date.now() - start > timeout) {
          clearInterval(interval);
          reject(new Error("Timeout waiting for dropdown."));
        }
      }, 100);
    });
  }

  // ─── Main Flow ─────────────────────────────────────────────────────────────
  // First, scroll to ensure every song is loaded.
  await scrollToBottom();

  // Query all menu triggers for the songs.
  const triggers = document.querySelectorAll(".dropdown-trigger.style-scope.ytmusic-menu-renderer");
  console.log(`Found ${triggers.length} menu trigger element(s).`);

  // Process each song one by one.
  for (let i = 0; i < triggers.length; i++) {
    try {
      // Click the menu trigger to open the dropdown.
      const trigger = triggers[i];
      trigger.click();
      console.log(`Clicked trigger for song ${i + 1}.`);
      
      // Wait a brief moment so the dropdown has time to open.
      await new Promise((r) => setTimeout(r, 300));
      
      // Wait for the dropdown element to appear.
      const dropdown = await waitForDropdown(5000);
      await new Promise((r) => setTimeout(r, 200)); // Allow dropdown items to render
      
      // Attempt to find the "Save to library" option.
      const itemsContainer = dropdown.querySelector("tp-yt-paper-listbox#items");
      let addSong = itemsContainer
        ? itemsContainer.querySelector("ytmusic-toggle-menu-service-item-renderer.style-scope.ytmusic-menu-popup-renderer")
        : null;
      
      if (addSong) {
        // Find the text element that has the label.
        const actualAddSong = addSong.querySelector("yt-formatted-string.text.style-scope.ytmusic-toggle-menu-service-item-renderer");
        if (actualAddSong && actualAddSong.innerText.trim() === "Save to library") {
          // Click the menu item to save the song.
          addSong.click();
          console.log(`Song ${i + 1}: Saved to library.`);
          await new Promise((r) => setTimeout(r, 300));
        } else {
          console.log(`Song ${i + 1}: "Save to library" option not found (possibly already saved).`);
        }
      } else {
        console.log(`Song ${i + 1}: Could not find the add song menu item.`);
      }
      
      // Close the dropdown by clicking outside.
      document.body.click();
      await new Promise((r) => setTimeout(r, 200));
    } catch (err) {
      console.error(`Error processing song ${i + 1}: ${err}`);
    }
  }
  console.log("Batch Save process complete!");
})();
```

## Contributing

Contributions, issues, or suggestions are welcome. Please open an issue or submit a pull request for improvements or bug fixes.

## License

This project is licensed under the [MIT License](LICENSE).
```
