# Ephemeris Data Repository

This repository contains ephemeris (celestial position) data files used by the **Fireballs in the Sky** mobile app. The app automatically syncs these files on startup to provide up-to-date celestial object positions for the planetarium feature.

**Important:** The json files must be valid json and pass javascripts JSON.parse() (This means no unnecessary trailing commas)

---

## Repository Structure

```
.
├── README.md (this file)
├── ephemeris_config.json
├── object-name-1.json
├── object-name-2.json
└── ...
```

---

## Configuration File: `ephemeris_config.json`

The `ephemeris_config.json` file is the entry point that lists all available ephemeris data files. The app checks this file first to determine which ephemeris datasets to sync.

### Structure

```json
{
  "ephemerisItems": [
    {
      "name": "unique-object-identifier",
      "path": "object-name.json"
    },
    {
      "name": "another-object",
      "path": "another-object.json"
    }
  ]
}
```

### Fields

- **`ephemerisItems`** (array, required): List of all ephemeris data files
  - **`name`** (string, required): Unique identifier for the celestial object. This is used as the storage key in the app. Use lowercase, hyphenated names (e.g., `"apophis"`, `"bennu"`, `"ceres"`).
  - **`path`** (string, required): Relative path from the repository root to the ephemeris data file (e.g., `"apophis_ephemeris.json"`).
- **`test`** (optional): Can be used to trigger an update to the config file's last modified date. (Any key name can be used, I didn't want to call it version because this specifically does not determine when the config is updated on the app).

### Example

```json
{
  "ephemerisItems": [
    {
      "name": "apophis",
      "path": "apophis_ephemeris.json"
    },
    {
      "name": "bennu",
      "path": "bennu_ephemeris.json"
    }
  ],
  "test": 8
}
```

---

## Ephemeris Data Files

Each ephemeris data file contains position data for a celestial object over time. These files are JSON arrays of position entries.
These files are generated via the script within the main project app's directory **`scripts/ephemeris_generator/gen_apophis.py`**

### Structure

```json
[
  {
    "timeUtc": "2024-01-01T00:00:00Z",
    "ra": 123.456,
    "dec": -45.678
  },
  {
    "timeUtc": "2024-01-01T01:00:00Z",
    "ra": 123.789,
    "dec": -45.712
  }
]
```

### Fields

- **`timeUtc`** (string, optional): ISO 8601 timestamp in UTC.
- **`ra`** (number, required): Right Ascension in degrees (0-360).
- **`dec`** (number, required): Declination in degrees (-90 to +90).

---

## How the App Syncs Data

The **Fireballs in the Sky** app uses the `GitHubSyncController` to automatically sync this data once on app startup:

1. **On App Startup**: The app checks the last modified date of `ephemeris_config.json` via the GitHub API.
2. **Config Comparison**: If the config file's last modified date has been changed since last sync, the app downloads and saves the new config file's data.
3. **Item Sync**: For each item in the config, the app:
   - Checks and compares the last modified date of the ephemeris data file with the locally stored version
   - Downloads and updates the locally stored version only if the file has been updated
4. **Storing**: All data is cached locally in the app's AsyncStorage for offline access.

---

## Adding or Updating Ephemeris Data

### Adding a New Object

1. **Create the ephemeris data file**:

   - Create a new JSON file
   - Name it descriptively (e.g., `apophis_ephemeris.json`, `bennu_ephemeris.json`)
   - Format it as a JSON array of ephemeris entries

2. **Update `ephemeris_config.json`**:

   - Add a new entry to the `ephemerisItems` array
   - Use a unique `name` identifier
   - Set the `path` to the relative path of your new file in the repo

3. **Commit and push**:
   - Commit both files
   - Push to the repository
   - The app will automatically sync the new data on the next startup

### Updating Existing Data

1. **Modify the ephemeris data file**:

   - Edit the JSON file with updated position data
   - Ensure the JSON is valid

2. **Commit and push**:
   - Commit the changes
   - Push to the repository
   - The app will detect the updated last modified date and sync the new data

### Removing an Object

1. **Remove from config**:

   - Remove the entry from `ephemeris_config.json`
   - Optionally delete the ephemeris data file

2. **Commit and push**:
   - The app will stop syncing the removed object
   - Note: The app currently doesn't automatically delete cached data for removed objects (this is a known TODO in the codebase)

---

## Generating Ephemeris Files

Ephemeris data files are generated using the `gen_apophis.py` script located in the main app's repository at `scripts/ephemeris_generator/`. This script queries NASA JPL Horizons to fetch celestial object positions and creates optimized sparse ephemeris files.

## Environment Variables (App Side)

The app requires these environment variables to sync from this repository, if you wish to use a different repo location these will have to be updated:

- `EXPO_PUBLIC_GITHUB_FILE_URL`: Base URL for fetching raw file contents
  - Example: `https://raw.githubusercontent.com/username/repo-name/branch/`
  - Becomes: `https://raw.githubusercontent.com/username/repo-name/branch/apophis_ephemeris.json`
- `EXPO_PUBLIC_GITHUB_COMMIT_URL`: Base URL for GitHub API commits endpoint
  - Example: `https://api.github.com/repos/username/repo-name/commits?sha=branch&path=`
  - Example: `https://api.github.com/repos/username/repo-name/commits?sha=branch&path=apophis_ephemeris.json`
