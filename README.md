# YTMusic-Save-To-Library

**YTMusic-Save-To-Library** is a lightweight JavaScript tool that batch-saves unsaved tracks from your YouTube Music playlists into your library. The script scrolls through your playlist, reveals hidden menu options, and automatically clicks the "Save to library" option for each song.

## Features

- **Batch Save:** Automatically add unsaved songs to your library.
- **Automatic Scrolling:** Loads the full playlist by scrolling.
- **Menu Interaction:** Simulates clicks to reveal and use YouTube Music's menu controls.
- **Easy Usage:** Run the script in your browser's Developer Console—no installation needed.

## How It Works

1. **Scroll:** The script scrolls to the bottom of the page until all songs are loaded.
2. **Menu Trigger:** It queries for each song's menu trigger element using the class `.dropdown-trigger.style-scope.ytmusic-menu-renderer`
3. **Dropdown & Save:** For each song, the script clicks the trigger, waits for the dropdown to appear, and then looks for a menu item labeled exactly "Save to library" (or "Add to library"). If found, it clicks it.
4. **Cleanup:** It closes the dropdown and moves on to the next song.

## Instructions

1. Open your YouTube Music playlist (or "Liked Music") page. (It's a good idea to scroll to the bottom manually first.)
2. Open your browser's Developer Console (F12, Ctrl+Shift+J, or Cmd+Option+J).
3. Paste the entire script (provided below) into the console and press Enter.
4. Watch the console logs for progress. The script will process each song and update your library accordingly.

## Usage

```javascript
(async function () {
  // ─── Helper: Scroll to the bottom so that all songs load ─────────────────────────────
  async function scrollToBottom(delay = 1500) {
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
      const startTime = Date.now();
      const interval = setInterval(() => {
        const dropdown = document.querySelector("ytmusic-menu-popup-renderer[slot='dropdown-content']");
        if (dropdown) {
          clearInterval(interval);
          resolve(dropdown);
        } else if (Date.now() - startTime > timeout) {
          clearInterval(interval);
          reject(new Error("Timeout waiting for dropdown."));
        }
      }, 100);
    });
  }

  // ─── Main Flow ───────────────────────────────────────────────────────
  // First, scroll down so all songs are loaded
  await scrollToBottom();

  // Get all song containers
  const songs = document.querySelectorAll("ytmusic-responsive-list-item-renderer");
  console.log(`Found ${songs.length} songs in the playlist.`);

  // Process each song one by one
  for (let i = 0; i < songs.length; i++) {
    const songContainer = songs[i];
    console.log(`Processing song ${i + 1} of ${songs.length}…`);

    // Simulate a mouseover on the song container so that the hidden menu becomes active
    songContainer.dispatchEvent(new MouseEvent("mouseover", { bubbles: true }));
    await new Promise((r) => setTimeout(r, 300));

    // Try to locate the menu placeholder within the song
    // In the current DOM, the menu is rendered in a <ytmusic-menu-renderer> element.
    let menuRenderer = songContainer.querySelector("ytmusic-menu-renderer");
    if (!menuRenderer) {
      console.log(`Song ${i + 1}: Menu renderer not found; skipping.`);
      continue;
    }

    // Look inside the placeholder for a button whose aria-label contains "menu"
    let menuButton = menuRenderer.querySelector("button[aria-label*='menu']");
    if (!menuButton) {
      console.log(`Song ${i + 1}: Menu button not found; skipping.`);
      continue;
    }

    // Click the menu button to bring up the dropdown options.
    menuButton.click();
    console.log(`Song ${i + 1}: Menu button clicked.`);
    
    // Wait for the dropdown menu to appear.
    let dropdown;
    try {
      dropdown = await waitForDropdown(5000);
    } catch (e) {
      console.log(`Song ${i + 1}: Dropdown did not appear; skipping.`);
      continue;
    }
    
    // Give a short delay to allow dropdown items to render.
    await new Promise((r) => setTimeout(r, 300));

    // Within the dropdown, search for the "Save to library" or "Add to library" option.
    const menuItems = dropdown.querySelectorAll("ytmusic-toggle-menu-service-item-renderer");
    let saveOption = null;
    for (let item of menuItems) {
      let textElem = item.querySelector("yt-formatted-string");
      if (textElem) {
        const text = textElem.innerText.trim();
        if (text === "Save to library" || text === "Add to library") {
          saveOption = item;
          break;
        }
      }
    }
    if (saveOption) {
      saveOption.click();
      console.log(`Song ${i + 1}: Song saved to library.`);
      await new Promise((r) => setTimeout(r, 500));
    } else {
      console.log(`Song ${i + 1}: "Save to library" option not found. Possibly already saved.`);
    }

    // Close the dropdown by clicking outside.
    document.body.click();
    await new Promise((r) => setTimeout(r, 300));
  }

  console.log("Batch Save process complete!");
})();

```

## Contributing

Contributions, issues, or suggestions are welcome. Please open an issue or submit a pull request for improvements or bug fixes.

## License

This project is licensed under the [MIT License](LICENSE).
