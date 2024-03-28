# Tutorials

This is our repo for draft tutorials. We'll review them here and then upload to our [Resources](https://talkjs.com/resources/) site, which uses the [Ghost](https://ghost.org/) platform.

## Notes for writers

- Clone the repo and make a new branch for your tutorial.
- Add your tutorial to a new folder.
- Write tutorials in Markdown – refer to Ghost's [reference guide](https://ghost.org/help/using-markdown/) for details.
- Add images and GIFs to the same folder. They'll need to be wrapped in some custom HTML to display correctly in Ghost, so paste this snippet in where you want the image to appear and fill out the captions and alt text. We'll add the image URL when we upload the tutorial.
  ```html
  <figure class="kg-image-card">
    <img class="kg-image" src="<URL>" alt="<ALT>"/>
    <figcaption>TEXT</figcaption>
  </figure>
  ```
- Create a PR when it's ready to review.

## Notes for reviewers uploading to Ghost

- Paste in the article in a Markdown block.
- Upload the images through Ghost's UI and paste the link in the `<URL>` placeholder.
