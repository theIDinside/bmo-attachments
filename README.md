# bmo-attachments

Download every attachment on a bugzilla.mozilla.org bug into a folder named after the bug number.

It uses the Bugzilla REST API with your personal API key, so it works on security bugs and other restricted bugs, as long as your account can already see them. It cannot see anything your account can't.

## Requirements

Python 3. Nothing else needs installing; it only uses the standard library.

## Setup

1. Create a Bugzilla API key (see [Creating a Bugzilla API key](#creating-a-bugzilla-api-key)).
2. Save the key to `~/.config/bmo-api-key`, which is where the script looks for it. Run the `cat` command below, paste the key, press Enter, then press Ctrl-D. Then run `chmod` so that only you can read the file:

   ```sh
   cat > ~/.config/bmo-api-key
   chmod 600 ~/.config/bmo-api-key
   ```

   You can also set the `BUGZILLA_API_KEY` environment variable instead; it takes precedence over the file.

3. Put the script somewhere on your `PATH`:

   ```sh
   cp bmo-attachments ~/.local/bin/
   ```

The key gives full access to your Bugzilla account, including any security bugs you can see. Don't commit it or share it. If it leaks, revoke it (see [Creating a Bugzilla API key](#creating-a-bugzilla-api-key)).

## Creating a Bugzilla API key

An API key lets a script act as your account without your password. Bugzilla only returns what your account can already see.

1. Log in to <https://bugzilla.mozilla.org>.
2. Open the API key preferences: click your name or avatar in the top-right corner, choose **Preferences**, then open the **API Keys** tab. You can also go straight to <https://bugzilla.mozilla.org/userprefs.cgi?tab=apikey>.
3. In the section for creating a new key, tick the checkbox to generate a new API key (if there is one), and enter a description such as `bmo-attachments`. The description is only there to help you recognize the key later.
4. Click **Submit Changes**.
5. The new key appears on the page as a long string of letters and digits. Copy it now and store it as described in [Setup](#setup). It may not be shown in full again; if you lose it, just create a new one.

To revoke a key, go back to the **API Keys** tab, tick **Revoked** next to it, and click **Submit Changes**. Scripts using that key stop working immediately.

## Usage

```sh
bmo-attachments 1234567                      # creates ./1234567/
bmo-attachments 1234567 -o ~/bugs            # creates ~/bugs/1234567/
bmo-attachments 1234567 --include-obsolete   # also fetch obsolete attachments
bmo-attachments 1234567 --include-phabricator  # also fetch Phabricator revision links
```

The bug folder will contain:

- `summary.txt`: the bug title and its URL.
- One file per attachment, named `<attachment-id>-<filename>`. The ID prefix stops two attachments with the same name (say, two `testcase.html`) from overwriting each other, and matches `attachment.cgi?id=<attachment-id>` on Bugzilla.

Obsolete attachments are skipped unless you pass `--include-obsolete`.

Phabricator patches are skipped unless you pass `--include-phabricator`. With it, each one is saved as a small `phabricator-D<nnn>-url.txt` file that just contains the revision URL.

### Meta bugs

If the bug has the `meta` keyword, the script also downloads every bug it depends on (the "Depends on" field), each into its own folder inside the meta bug's folder. Depended-on bugs that are themselves meta bugs are followed the same way, so the folders nest:

```
1234567/                 # the meta bug
├── summary.txt
├── 1111111/             # a bug it depends on
│   └── summary.txt
└── 2222222/             # another meta bug it depends on
    ├── summary.txt
    └── 3333333/
        └── summary.txt
```

Each bug is downloaded only once, even if several meta bugs depend on it. If a depended-on bug can't be fetched (for example, you don't have access to it), the script prints a warning and carries on with the rest.

## Notes

- Re-running the command overwrites files that are already in the folder.
- If the bug doesn't exist, you can't see it, or your key is invalid, the script stops and prints the error message from Bugzilla.
