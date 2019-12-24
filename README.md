# managed-wordpress-deploy

Action for deploying a compiled WordPress theme to a managed host like WP Engine

## Inputs

### remote-host

Address of the remote server we're going to depoy onto

## Outputs

### `time`

ok, this is a thing

---

# Working notes

Use find to collect files for replacement, something like this:

```
  find iop-gip-backup/ -type f -regextype posix-extended -regex ".*(dist|vendor/composer).*" -exec sed -i 's/-0_0_66/-backup/g' {} +
```

Env vars we'll need to know:

- **WP_THEME_DIR** : `~/sites/gip7b70f58129a/wp-content/themes`
- **WP_HOST** : `gip7b70f58129a.ssh.wpengine.net`
- **SSH_KEY** : _{stored in GitHub secrets}_
- **SSH_USER** : `gip7b70f58129a` (do not assume this matches anything else)

Should also modify the duplicated theme's style.css file and change the name. It would be nice if the backed up theme also included the date there so we could see when it was created from the WordPress theme admin screen.

Do we have an organization-wide SSH key? would that be better than several individual keys?

Sed to extract version from theme/style.css:
`sed -n 's/Version:\s+\([0-9.]+\)$/\1/' style.css`

Date string: `date "+%Y%m%dT%H%M"`

Remote RSYNC command `ssh gip7b70f58129a@gip7b70f58129a.ssh.wpengine.net -i ~/.ssh/id_rsa_wpengine -t "rsync -av ~/sites/gip7b70f58129a/wp-content/themes/iop-gip-0_0_66 ~/sites/gip7b70f58129a/wp-content/themes/iop-gip-b2"`

## Things we know

1. Name/slug of theme being deployed (package.json from checkout)

## Things we should not assume

1. theme may or may not be installed (use tmp-name for transfer)
2. wp-cli may or may notbe available (likely not)
3. Old theme may not be properly formed/described (may not have a version, etc)

## Possible workflow?

Normal development produces many commits to GitHub, once we decide we're ready to deploy, click "New Release" to trigger the following process, hopefully all through GitHub Actions.

- [x] Build zip of theme from GitHub new Release
- [ ] scp new theme to `WP_HOST` using a temp-named dir (or /tmp?) (last commit hash?)
- [ ] rsync existing theme to timestamped backup folder
- [ ] run find/sed replacement on backup folder (so it's funcitonal, mostly unique to our builds)
- [ ] rename tmp-named new them to real name
