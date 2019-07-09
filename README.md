This is a fork of [Medium Editor Insert Plugin](https://github.com/orthes/medium-editor-insert-plugin) for use in Gumroad’s app.

## Why fork?
Gumroad uses [Medium Editor](https://github.com/yabwe/medium-editor) for our rich text editor, and this plugin enables the ability to embed images and oembedly links within it.

- See our implementation of medium editor in Product descriptions, Update messages, and Workflow email messages.
- [See demo of this plugin](http://orthes.github.io/medium-editor-insert-plugin)

In implementing this plugin into our app, we needed ways to:
- use our existing Flight components to handle image upload to S3 (WithFileUpload)
- make it play well with FlightJS itself
- make it end-to-end testable w/ capybara spec tests

Additionally:
- we wanted to add this plugin to our repo as an NPM package instead of hard-importing JS files
- we would like to extend this plugin to introduce new extension types (e.g. CTA buttons)
- at the time of this writing (May 2019), the original plugin has not been updated for 3 years.

While not very ideal, our only option was to fork this plugin.

## How do I make updates to this plugin?

1) yarn install
1) Make changes to /src/ files
3) Run "grunt js" to compile files for distro
4) Open a PR and request a codereview from someone in #engineering-chatter
5) Once merged and ready to be updated, bump up the version number in package.json, git tag [VERSION_NUMBER], git push origin --tags
6) Update the version number in gumroad/web's package.json

## What are the main changes we had to make?

- We do not use this plugin's native fileupload() methods for anything
- When an image is selected, this plugin emits a uiNeedsToUploadRichTextImage event we listen to in our flight component (WithRichTextImageUpload). This includes a copy of the file object we can use to trigger the actual S3 upload.
- This plugin adds a unique image identifier around the image once uploaded using the hashCode() method within it. This method is available via gumroad/web's string utils. This is so that we can target the image and replace the binary data with the actual S3 URL once uploaded within our Flight component in gumroad/web.

```
var tempImageClassName = (data.files[0].name + data.files[0].size).hashCode();
...
$place.addClass(‘contains-image-‘ + tempImageClassName);
```
- The plugin looks for a skip-dialog class around the file upload input before triggering the file upload modal. This is so that we can override triggering the native file upload modal in capybara tests; without doing this, the open file modal would block everything. See example in updates_spec in gumroad/web for how we test this.
```
      find(".js-update-message").base.send_keys(:enter)
      page.execute_script("$('.medium-insert-buttons-show').click();")
      page.execute_script("$(\"#medium-insert-file-selector\").addClass('skip-dialog')")
      page.execute_script("$('.medium-insert-action').click();")
      page.attach_file("medium-insert-file-selector", Rails.root.join("spec/sample_data/test.jpg"))

      Timeout.timeout(Capybara.default_max_wait_time) do
        loop until page.evaluate_script("$(\".medium-insert-images-progress\").length == 0;")
      end

```
- Remove jquery.sortage (unnecessary dependency)
- Hide toolbar options on images (unnecessary, was very buggy)

Notable components to look at in gumroad/web:
- WithRichText
- WithRichTextImageUpload

## Version history

2.6.6 -- Fix the ability to delete an embed from the current caret position using the delete key
2.6.5 -- Fix formatting for oembed captions
2.6.4 -- Add hashCode util function and support onImageSelect callback on context.
2.6.3 -- recalibrate custom css changes into this repo
2.6.2 -- remove duplication of input field selector in buttons panel
2.6.1 -- add a disable toolbar option for plugins and fix caret placement after oembed bug
2.6.0 -- initial updates for Gumroad's implementation of inline images
