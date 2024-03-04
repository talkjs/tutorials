# Tutorials

This is our repo for draft tutorials. We'll review them here and then upload to our [Resources](https://talkjs.com/resources/) site, which uses the [Ghost](https://ghost.org/) platform.

## Notes for writers

- Clone the repo and make a new branch for your tutorial
- Add your tutorial to a new folder
- Write tutorials in Markdown – refer to Ghost's [reference guide](https://ghost.org/help/using-markdown/) for details
- Add images and GIFs to the same folder and mark where you want them to appear in the tutorial text (for example, with `!! <file_name>.jpg`)
- Include captions and alt text for images
- Create a PR when it's ready to review

## Notes for reviewers uploading to Ghost

Paste in the article in a Markdown block.

Images need some custom HTML to display correctly, so paste this in where you want the image to appear:

```html
<figure class="kg-image-card">
  <img class="kg-image" src="<URL>" alt="<ALT>"/>
  <figcaption>TEXT</figcaption>
</figure>
```

Then upload the image through Ghost's UI and paste the link in the `<URL>` placeholder. Fill in the caption and alt text.
