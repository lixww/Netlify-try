+++
title = "Blog workflow"
date = 2024-06-13
[taxonomies]
  tags = ["workflow"]
+++


In case I forget one day.

Inspired by [Limboy](https://limboy.me/posts/my-blog-system/).

# Components

- Zola
- Github
- Netlify

Blogs are store in a private repository on my Github. Blogs architecture is created using Zola. Netlify is used to deploy my zola-blogs on the internet and generate blog sites under Netlify’s domain.

So the result is: [**https://vvone.netlify.app**](https://vvone.netlify.app/)

Sites overview on Netlify.

# Workflow

**Start From Zero**

Create a private repository on Github. Install Zola and use Zola-init under the git repository. Follow the Netlify set-up and link the git repository with the Netlify project. Follow the deploy guide. Visit the my zola-site from browser.

**Start Now**

Now add blog under content/ , git commit and push the modification. Since `git push` is link to the `zola build` command on Netlify, each git push (to the main branch) will start a deployment immediately. Then new modification of the blog can be accessed through browser.

# By the way

~~I bought a Vultr machine, which is useless now. Do I need it? Really cost me. 🙉~~

Update on 2024/1/30, It’s already been closed on 2023/10/5. :)

# Theme

From Zola.

- [Bear](https://www.getzola.org/themes/bearblog/)
- [anemone](https://www.getzola.org/themes/anemone/)