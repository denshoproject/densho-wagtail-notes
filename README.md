# Wagtail Notes

Add MediaWiki-style footnotes functionality to your Wagtail project.

## ⚡ Quick start

Add `wagtail_notes` to `INSTALLED_APPS`:

```python
# settings.py

INSTALLED_APPS = [
    # ...
    "wagtail_notes",
    # ...
]
```

Update your models to import the `footnotes` module:
``` python
from wagtail_notes import footnotes
```

Add a `footnotes` field to your model:
``` python
    ...
    footnotes = RichTextField(blank=True, null=True)
    ...
```

Register model fields to be scanned for footnotes:
``` python
FOOTNOTE_FIELDS = {
    'richtextfields': ['description'],
    'streamfields': ['body'],
}
```

Add hooks (to a model named `Article`) so footnotes will be collected after editing and before serving:
``` python
@hooks.register('after_create_page')
def do_after_page_create(request, page):
    if isinstance(page, Article):
        return footnotes.Footnotary.update_footnotes(
            page, FOOTNOTE_FIELDS, request
        )

@hooks.register('after_edit_page')
def do_after_page_edit(request, page):
    if isinstance(page, Article):
        return footnotes.Footnotary.update_footnotes(
            page, FOOTNOTE_FIELDS, request
        )

@hooks.register('before_serve_page')
def prep_footnotes(page, request, serve_args, serve_kwargs):
    if isinstance(page, Article):
        return footnotes.Footnotary.prep_footnotes(
            page, FOOTNOTE_FIELDS, request
        )
```

Make and run migrations:

```shell
python manage.py makemigrations
python manage.py migrate
```

### Showing footnotes in page templates

Add the following
``` python
  {% if article.footnotes %}
  <h3>Footnotes</h3>
  <div class="footnotes">
    {{ article.footnotes | safe }}
  </div><!-- footnotes -->
  {% endif %}
```

### Using footnotes in `RichTextField`s and `StreamField`s

Write your footnote text within the body of your article, surrounded by `<ref>` and `</ref>` tags.  For example:
``` html
This is regular paragraph text that needs a citation or a footnote.<ref>This is the text of the footnote.</ref>
```


## Under the hood/bonnet

When you save a revision or publish, the registered fields will be scraped and the contents of `<ref>` tags extracted.  The `<ref>` tags and their contents will be copied to the `footnotes` field (overwriting whatever is currently there), while text in your registered fields will be left as it is.

When displaying your page, `<ref>` tags in the `footnotes` field will be turned into `<a name>` tags in a numbered list.  `<ref>` tags in the registered fields will be converted into `<a href>` tags that point to the corresponding footnotes; their contents will be replaced with a number.

It is important that users are not able to modify the `footnotes` field in the editor.


## Contributing

All contributions are welcome!


### Install

To make changes to this project, first clone this repository:

```sh
git clone git@github.com:denshoproject/densho-wagtail-notes.git
cd densho-wagtail-footnotes
```
