# bmo-attachments

Download every attachment on a bugzilla.mozilla.org bug into a folder named after the bug number.

It uses the Bugzilla REST API with your personal API key, so it works on security bugs and other restricted bugs, as long as your account can already see them. It cannot see anything your account can't.

## Requirements

Python 3. Nothing else needs installing; it only uses the standard library.

## Setup

1. Create an API key at <https://bugzilla.mozilla.org/userprefs.cgi?tab=apikey>. Type a description (e.g. `bmo-attachments`) and click **Submit Changes**. The key is only shown once, so copy it right away.
2. Store the key where only you can read it:

   ```sh
   echo 'YOUR_API_KEY' > ~/.config/bmo-api-key
   chmod 600 ~/.config/bmo-api-key
   ```

   You can also set the `BUGZILLA_API_KEY` environment variable instead; it takes precedence over the file.

3. Put the script somewhere on your `PATH`:

   ```sh
   cp bmo-attachments ~/.local/bin/
   ```

The key gives full access to your Bugzilla account, including any security bugs you can see. Don't commit it or share it. If it leaks, revoke it on the same preferences page.

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

## Notes

- Re-running the command overwrites files that are already in the folder.
- If the bug doesn't exist, you can't see it, or your key is invalid, the script stops and prints the error message from Bugzilla.
