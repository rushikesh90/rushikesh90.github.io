---
layout: single
title: "Setting Up Your Academic Homepage with GitHub Pages: A Beginner's Guide"
date: 2026-06-03
categories: [tutorial, github-pages, academic]
tags: [github-pages, academicpages, jekyll, tutorial]
toc: true
---

Creating a professional academic homepage is essential for showcasing your research, publications, and academic achievements. GitHub Pages, combined with the popular [academicpages](https://github.com/academicpages/academicpages.github.io) theme, provides a free, easy-to-maintain solution. This tutorial walks you through the entire process step-by-step.

## Prerequisites

- A GitHub account
- Basic understanding of Git and GitHub (we'll cover the essentials)
- No prior web development experience required

## Step 1: Fork the AcademicPages Repository

1. Go to the [AcademicPages repository](https://github.com/academicpages/academicpages.github.io)
2. Click the "Fork" button in the top-right corner to create a copy in your GitHub account
3. Wait for the fork to complete (it usually takes a few seconds)

![Fork button](https://docs.github.com/assets/images/help/repository/fork-button.jpg)

## Step 2: Rename Your Repository

GitHub Pages sites follow a specific naming convention: `<username>.github.io`

1. Navigate to your forked repository (it should be at `https://github.com/yourusername/academicpages.github.io`)
2. Click on "Settings" in the repository menu
3. Under the "General" tab, find the "Repository name" field
4. Change the name from `academicpages.github.io` to `yourusername.github.io` (replace `yourusername` with your actual GitHub username)
5. Click "Rename" to save the changes

> **Important**: After renaming, GitHub will automatically serve your site at `https://yourusername.github.io`

## Step 3: Verify GitHub Pages is Enabled

1. In your repository settings, scroll down to the "GitHub Pages" section
2. Under "Source", ensure it's set to deploy from the `main` branch (or `master` if that's your default branch)
3. The folder should be `/ (root)`
4. You should see a message indicating your site is published at `https://yourusername.github.io`

![GitHub Pages settings](https://docs.github.com/assets/images/help/repository/pages-settings.png)

## Step 4: Customize Your Site Configuration

The `_config.yml` file controls your site's settings. We'll customize it to reflect your information.

### Clone Your Repository Locally

To make changes, you'll need to clone your repository to your local machine:

```bash
git clone https://github.com/yourusername/yourusername.github.io.git
cd yourusername.github.io
```

### Edit _config.yml

Open `_config.yml` in your favorite text editor and update the following sections:

#### Site Settings
```yaml
title: Your Name          # Your full name
email: your.email@example.com  # Your email address
description: >-           # A brief description of yourself
  Your research interests or professional bio
baseurl: ""               # Leave empty for user sites
url: "https://yourusername.github.io"  # Your site URL
```

#### Author Information
```yaml
author:
  name   : "Your Name"
  avatar : "/assets/images/bio-photo.jpg"  # We'll add this later
  bio    : "Your brief biography"
  location: "Your Location"
  email  : "your.email@example.com"
  links:
    - label: "Email"
      icon: "fas fa-fw fa-envelope-square"
      url: "mailto:your.email@example.com"
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/yourusername"
    - label: "Google Scholar"
      icon: "ai ai-google-scholar"
      url: "https://scholar.google.com/citations?user=your-id"
```

#### Site Features
Toggle features on/off by setting to `true` or `false`:
```yaml
# Whether to show a "Contact" link in the menu
menu:
  - title: "About"
    url: "/about/"
  - title: "Blog"
    url: "/blog/"
  - title: "Publications"
    url: "/publications/"
  - title: "CV"
    url: "/cv/"
  - title: "Contact"
    url: "/contact/"
```

### Add Your Bio Photo

1. Place your photo in the `assets/images/` directory
2. Name it `bio-photo.jpg` (or update the path in `_config.yml` accordingly)
3. Recommended size: 400x400 pixels

## Step 5: Add Your Content

AcademicPages uses Markdown for content. Let's customize the main sections:

### About Me Page

Edit `_pages/about.md`:
```markdown
---
permalink: /about/
title: "About"
author_profile: true
---

Write a brief introduction about yourself here. Include your current position, research interests, and academic background.

You can also add a list of your current affiliations:

- **Position**: Assistant Professor, Department of Computer Science
- **Institution**: Your University
- **Address**: Department Address
- **Email**: your.email@university.edu
```

### Publications Page

Edit `_pages/publications.md`:
```markdown
---
permalink: /publications/
title: "Publications"
author_profile: true
---

{% include base_path %}

You can list your publications manually or use a script to generate them from a BibTeX file. For simplicity, we'll add them manually.

### Journal Articles

1. **Author1**, Author2, Author3. "Paper Title." *Journal Name*, vol. 1, no. 1, pp. 1-10, Year.
   [DOI](https://doi.org/10.1234/example) | [PDF](link-to-pdf)

### Conference Papers

1. Author1, **Author2**, Author3. "Conference Paper Title." In *Proceedings of Conference Name*, pp. 100-110, Year.
   [DOI](https://doi.org/10.1234/example) | [PDF](link-to-pdf)
```

### CV Page

Edit `_pages/cv.md`:
```markdown
---
permalink: /cv/
title: "CV"
author_profile: true
---

{% include base_path %}

You can embed a PDF version of your CV or write it in Markdown.

<iframe src="/assets/files/cv.pdf" width="800" height="1000" style="border: none;"></iframe>
```

Place your CV PDF in the `assets/files/` directory.

## Step 6: Test Your Site Locally (Optional but Recommended)

To preview changes before pushing:

1. Install Ruby and Bundler if you haven't already
2. Install Jekyll and dependencies:
   ```bash
   bundle install
   ```
3. Start the local server:
   ```bash
   bundle exec jekyll serve
   ```
4. Open `http://localhost:4000` in your browser

## Step 7: Push Changes to GitHub

After making your changes:

```bash
git add .
git commit -m "Initial setup of academic homepage"
git push origin main
```

GitHub will automatically rebuild your site. Visit `https://yourusername.github.io` to see your live site!

## Step 8: Customize Further (Optional)

### Change the Theme Color

Edit `assets/css/main.scss` to modify the primary color:
```scss
$brand-color: #your-color-code;  // Replace with your preferred color
```

### Add Google Analytics

1. Get your Google Analytics tracking ID
2. Add to `_config.yml`:
   ```yaml
   google_analytics: "UA-XXXXXXXXX-X"
   ```

### Enable Comments

AcademicPages supports various comment systems. To enable Utterances:
1. Install the Utterances app on your GitHub repository
2. Add to `_config.yml`:
   ```yaml
   comments:
     provider: "utterances"
     utterances:
       theme: "github-light"  // or "github-dark"
       issue_term: "pathname"
   ```

## Troubleshooting Common Issues

### Site Not Updating
- Ensure you're pushing to the correct branch (`main` or `master`)
- Check GitHub Actions tab for build errors
- Wait a few minutes for GitHub Pages to rebuild

### Images Not Loading
- Verify image paths are correct
- Ensure images are committed to the repository
- Use relative paths like `/assets/images/your-image.jpg`

### Links Broken
- Check that your `url` and `baseurl` in `_config.yml` are correct
- For user sites, `baseurl` should be empty

## Conclusion

Congratulations! You've successfully set up your academic homepage using GitHub Pages and the AcademicPages theme. Your site is now live at `https://yourusername.github.io` and ready to showcase your academic work.

Remember to:
- Regularly update your publications and achievements
- Keep your CV current
- Periodically check for broken links
- Consider adding a blog section to share your research insights

Your academic homepage is a living document—update it as your career progresses. Happy publishing!

## References

- [AcademicPages GitHub Repository](https://github.com/academicpages/academicpages.github.io)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [Jekyll Documentation](https://jekyllrb.com/docs/home/)