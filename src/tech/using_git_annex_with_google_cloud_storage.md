# Using Git Annex with Google Cloud Storage

Dec 11, 2020.  We are trying to use `git annex` to store large
files in a Google Cloud Storage bucket.

## Set Up rclone

Let's set up `rclone` so that it's linked to our Google Cloud
account.

- Install [git annex](https://git-annex.branchable.com/install/).
- Install [rclone](https://rclone.org/install/).
- Set up rclone [for Google Cloud Storage](https://rclone.org/googlecloudstorage/).

You will be prompted in your browser to allow `rclone` access
to your Google Cloud account.

Once you complete these steps, you will be able to see the
contents of your cloud storage bucket:

```text
$ rclone lsd my-remote:
-1 2020-12-11 16:00:02        -1 some-project-name
```

## Set Up git-annex

Now let's set up `git-annex` so that it can use the newly-defined
`rclone` remote.

You need the bash script from [git-annex-remote-rclone](https://github.com/DanielDent/git-annex-remote-rclone) to tie everything together:

```sh
curl https://raw.githubusercontent.com/DanielDent/git-annex-remote-rclone/master/git-annex-remote-rclone --output ~/bin/git-annex-remote-rclone
chmod 755 ~/bin/git-annex-remote-rclone
```

To configure `git-annex` with a 5MB chunk size:

```sh
BUCKET=some-bucket-name RCLONE_REMOTE=rclone-remote-name git annex initremote gcs-ruins type=external externaltype=rclone target=$RCLONE_REMOTE prefix=$BUCKET chunk=5MiB encryption=shared mac=HMACSHA512 rclone_layout=lower
```