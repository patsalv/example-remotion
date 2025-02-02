# Remotion with Stoat Video Preview

This is a fork of the [`remotion-dev/template-helloworld`](https://github.com/remotion-dev/template-helloworld) repo to demo how to use Stoat to preview your Remotion video. Also see [this tutorial](https://docs.stoat.dev/docs/tutorials/preview-remotion) for details.

1. Install the [Stoat App](https://github.com/apps/stoat-app) for the repo.
2. Create a GitHub workflow for preview. An example can be found in [`.github/workflows/preview-video.yml`](.github/workflows/preview-video.yml).

    ```yaml
    name: Preview video
    on:
     push:
       branches:
         - main
     pull_request:
       branches:
         - main
    jobs:
     render:
       name: Render video
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - uses: actions/setup-node@v3
           with:
             node-version: '16'
             cache: npm
         - run: sudo apt update
         - run: sudo apt install ffmpeg
         - run: npm i
         - name: Generate video
           run: npm run build
         - name: Generate gif
           run: ffmpeg -i out/video.mp4 -vf "fps=10,scale=640:-1" -pix_fmt rgb24 out/video.gif
         - name: Run Stoat Action
           uses: stoat-dev/stoat-action@v0
           if: always()
    ```

3. Add the Stoat config in [`.stoat/config.yaml`](.stoat/config.yaml).

    ```yaml
    ---
    version: 1
    enabled: true
    comment_template_file: .stoat/comment.hbs
    plugins:
      static_hosting:
        video_preview:
          # This file is generated by remotion in the
          # "Generate video" step of the GitHub workflow.
          path: out/video.mp4
        gif_preview:
          # This file is generated from the remotion video in the
          # "Generate gif" step of the GitHub workflow.
          path: out/video.gif
    ```

4. Add the Stoat template in [`.stoat/comment.hbs`](.stoat/comment.hbs).

    ```handlebars
    ## Preview

    [![preview]({{ plugins.static_hosting.gif_preview.link }})]({{ plugins.static_hosting.video_preview.link }})
    
    (Click the GIF go to the full video.)
    ```
   
    The template file path must match the `comment_template_file` field in the Stoat config.

5. That's it. Now each time you make changes to the repo and opens a pull request, a comment will show up to provide a preview of the generated video like this:

    > [![preview](https://stoat-dev--example-re-cf7a--2bc0178--gif-preview.stoat.page/video.gif)](https://stoat-dev--example-re-cf7a--2bc0178--video-preview.stoat.page/video.mp4)
    >
    > (Click the GIF go to the full video.)
    
    Check out this [pull request](https://github.com/stoat-dev/example-remotion/pull/2) for details.
