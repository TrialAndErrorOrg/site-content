# site-content

What editors change on [trialanderror.org](https://trialanderror.org), edited through
[trialanderror.org/admin](https://trialanderror.org/admin). The site itself is `apps/site` in
[TrialAndErrorOrg/trialanderror](https://github.com/TrialAndErrorOrg/trialanderror).

| Folder | What | In /admin |
| --- | --- | --- |
| `pages/` | Pages built from sections. `index.yaml` is the homepage, `about.yaml` is /about | Pages |
| `settings/site.yaml` | Site name, blog and journal texts, newsletter, analytics | Settings → Site |
| `settings/navigation.yaml` | Header, journal and footer menus, social links | Settings → Menus |
| `uploads/` | Images and files, served at `/uploads/...` | Media |

Every save in /admin is a commit here. On each push to `main`, `.github/workflows/rebuild.yml`
asks the site to rebuild, which takes a few minutes.

The fields each file may have are defined in the site: `src/content.config.ts` (pages),
`src/lib/settings.ts` (settings) and `src/admin/config.yml` (the editing interface). A file that
doesn't match fails the build, and the live site stays as it was.

## Editing by hand

Change a file and push to `main`. To see it first, check this repository out next to
`trialanderror` and run the site there (`pnpm turbo run dev --filter=site`): it reads the files from
here.
