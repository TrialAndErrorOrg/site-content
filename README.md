# site-content

What editors change on [trialanderror.org](https://trialanderror.org), edited through
[trialanderror.org/admin](https://trialanderror.org/admin). The site itself is `apps/site` in
[TrialAndErrorOrg/trialanderror](https://github.com/TrialAndErrorOrg/trialanderror).

| Folder | What | In /admin |
| --- | --- | --- |
| `blog/` | Blog posts, Markdown with the details on top | Blog |
| `people/` | Team members and blog authors | People |
| `categories/` | Blog categories | Blog categories |
| `pages/` | Pages built from sections. `index.yaml` is the homepage, `about.yaml` is /about | Pages |
| `settings/site.yaml` | Site name, blog and journal texts, newsletter, analytics | Settings → Site |
| `settings/navigation.yaml` | Header, journal and footer menus, social links | Settings → Menus |
| `uploads/` | Images and files, served at `/uploads/...` | Media |

Saving in /admin makes a draft: a `cms/...` branch with a pull request. Publishing it from the
Workflow view merges it into `main`. On each push to `main`, `.github/workflows/rebuild.yml` asks
the site to rebuild, which takes a few minutes.

Uploads are public the moment they're saved, and stay in this repository's history after they're
deleted. Removing one for good means rewriting the history (`git filter-repo`) and force-pushing.

The blog, people and their images came from Strapi on 2026-10-14
(`apps/site/scripts/blog-to-content.ts` in the site's repository).

The fields each file may have are defined in the site: `src/content.config.ts` (pages),
`src/lib/settings.ts` (settings) and `src/admin/config.yml` (the editing interface). A file that
doesn't match fails the build, and the live site stays as it was.

## Editing by hand

Change a file and push to `main`. To see it first, check this repository out next to
`trialanderror` and run the site there (`pnpm turbo run dev --filter=site`): it reads the files from
here.
