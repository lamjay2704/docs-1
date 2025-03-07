# Assets Fields

Assets fields allow you to relate [assets](assets.md) to other elements.

## Settings

Assets fields have the following settings:

- **Restrict uploads to a single folder?** – Whether file uploads/relations should be constrained to a single folder.

  If enabled, the following setting will be visible:

  - **Upload Location** – The location that files dragged directly onto the field should be saved in.

  If disabled, the following settings will be visible:

  - **Sources** – Which asset volumes (or other asset index sources) the field should be able to relate assets from.
  - **Default Upload Location** – The default location that files dragged directly onto the field should be saved in.

- **Restrict allowed file types?** Whether the field should only be able to upload/relate files of a certain type(s).
- **Limit** – The maximum number of assets that can be related with the field at once. (Default is no limit.)
- **View Mode** – How the field should appear for authors.
- **Selection Label** – The label that should be used on the field’s selection button.

### Multi-Site Settings

On multi-site installs, the following settings will also be available (under **Advanced**):

- **Relate assets from a specific site?** – Whether to only allow relations to assets from a specific site.

  If enabled, a new setting will appear where you can choose which site.

  If disabled, related assets will always be pulled from the current site.

- **Manage relations on a per-site basis** – Whether each site should get its own set of related assets.

### Dynamic Subfolder Paths

Subfolder paths defined by the **Upload Location** and **Default Upload Location** settings can optionally contain Twig tags (e.g. `news/{{ slug }}`).

Any properties supported by the source element (the element that has the Assets field) can be used here.

::: tip
If you want to include the entry’s ID or UID in a dynamic subfolder path, use `{canonicalId}` or `{canonicalUid}` rather than `{id}` or `{uid}`, so the source entry’s ID or UID is used rather than the revision / draft’s.
:::

::: tip
If you are creating the Assets field within a [Matrix field](matrix-fields.md), the source element is going to be the Matrix block, _not_ the element that the Matrix field is being created on.

So if your Matrix field is attached to an entry, and you want to output the entry ID in your dynamic subfolder path, use `owner.id` rather than `id`.
:::

::: warning
If the rendered subfolder path ends up blank, or contains a leading or trailing forward slash (e.g. `foo/`) or an empty segment (e.g. `foo//bar`), Craft will interpret that as a sign that a variable in the subfolder template couldn’t be resolved successfully, and the path will be considered invalid. If you are intentionally outputting a blank segment, output `:ignore:`. For example, if you want to output the first selected category, or nothing if there isn’t one, do this:

```twig
{{ categoriesFieldHandle.one().slug ?? ':ignore:' }}
```
:::

## The Field

Assets fields list all the currently-related assets, with a button to select new ones.

Choosing **Add an asset** will bring up a modal window where you can find and select additional assets, as well as upload new ones.

::: tip
You can also upload assets by dragging files directly onto the assets field or modal window.
:::

### Inline Asset Editing

When you double-click on a related asset, a HUD will appear where you can edit the asset’s title and custom fields, and launch the Image Editor (if it’s an image).

::: tip
You can choose which custom fields should be available for your assets from **Settings** → **Assets** → **[Volume Name]** → **Field Layout**.
:::

## Development

### Querying Elements with Assets Fields

When [querying for elements](element-queries.md) that have an Assets field, you can filter the results based on the Assets field data using a query param named after your field’s handle.

Possible values include:

| Value | Fetches elements…
| - | -
| `':empty:'` | that don’t have any related assets.
| `':notempty:'` | that have at least one related asset.
| `100` | that are related to the asset with an ID of 100.
| `[100, 200]` | that are related to an asset with an ID of 100 or 200.
| `[':empty:', 100, 200]` | with no related assets, or are related to an asset with an ID of 100 or 200.
| `['and', 100, 200]` | that are related to the assets with IDs of 100 and 200.
| an [Asset](craft3:craft\elements\Asset) object | that are related to the asset.
| an [AssetQuery](craft3:craft\elements\db\AssetQuery) object | that are related to any of the resulting assets.

::: code
```twig
{# Fetch entries with a related asset #}
{% set entries = craft.entries()
  .myFieldHandle(':notempty:')
  .all() %}
```
```php
// Fetch entries with a related asset
$entries = \craft\elements\Entry::find()
    ->myFieldHandle(':notempty:')
    ->all();
```
:::

### Working with Assets Field Data

If you have an element with an Assets field in your template, you can access its related assets using your Assets field’s handle:

::: code
```twig
{% set query = entry.myFieldHandle %}
```
```php
$query = $entry->myFieldHandle;
```
:::

That will give you an [asset query](assets.md#querying-assets), prepped to output all the related assets for the given field.

To loop through all the related assets, call [all()](craft3:craft\db\Query::all()) and then loop over the results:

::: code
```twig
{% set relatedAssets = entry.myFieldHandle.all() %}
{% if relatedAssets|length %}
  <ul>
    {% for rel in relatedAssets %}
      <li><a href="{{ rel.url }}">{{ rel.filename }}</a></li>
    {% endfor %}
  </ul>
{% endif %}
```
```php
$relatedAssets = $entry->myFieldHandle->all();

if (count($relatedAssets)) {
    foreach ($relatedAssets as $rel) {
        // do something with $rel->url and $rel->filename
    }
}
```
:::

::: warning
When using `asset.url` or `asset.getUrl()`, the asset’s source volume must have **Assets in this volume have public URLs** enabled and a **Base URL** setting. Otherwise, the result will always be empty.
:::

If you only want the first related asset, call [one()](craft3:craft\db\Query::one()) instead and make sure it returned something:

::: code
```twig
{% set rel = entry.myFieldHandle.one() %}
{% if rel %}
  <p><a href="{{ rel.url }}">{{ rel.filename }}</a></p>
{% endif %}
```
```php
$rel = $entry->myFieldHandle->one();
if ($rel) {
    // do something with $rel->url and $rel->filename
}
```
:::

If you need to check for related assets without fetching them, you can call [exists()](craft3:craft\db\Query::exists()):

::: code
```twig
{% if entry.myFieldHandle.exists() %}
  <p>There are related assets!</p>
{% endif %}
```
```php
if ($entry->myFieldHandle->exists()) {
    // do something with related assets
}
```
:::

You can set [parameters](assets.md#parameters) on the asset query as well. For example, to ensure that only images are returned, you can set the [kind](assets.md#kind) param:

::: code
```twig
{% set relatedAssets = clone(entry.myFieldHandle)
  .kind('image')
  .all() %}
```
```php
$relatedAssets = (clone $entry->myFieldHandle)
    ->kind('image')
    ->all();
```
:::

::: tip
It’s always a good idea to clone the asset query using the [clone()](./dev/functions.md#clone) function before adjusting its parameters, so the parameters don’t have unexpected consequences later on in your template.
:::

### Saving Assets Fields

If you have an element form, such as an [entry form](https://craftcms.com/knowledge-base/entry-form), that needs to contain an Assets field, you will need to submit your field value as a list of asset IDs in the order you want them to be related.

For example, you could create a list of checkboxes for each of the possible relations:

```twig
{# Include a hidden input first so Craft knows to update the existing value
   if no checkboxes are checked. #}
{{ hiddenInput('fields[myFieldHandle]', '') }}

{# Get all of the possible asset options #}
{% set possibleAssets = craft.assets()
  .volume('siteAssets')
  .kind('image')
  .orderBy('filename ASC')
  .withTransforms([{ width: 100, height: 100 }])
  .all() %}

{# Get the currently related asset IDs #}
{% set relatedAssetIds = entry is defined
  ? entry.myFieldHandle.ids()
  : [] %}

<ul>
  {% for possibleAsset in possibleAssets %}
    <li>
      <label>
        {{ input(
          'checkbox',
          'fields[myFieldHandle][]',
          possibleAsset.id,
          { checked: possibleAsset.id in relatedAssetIds }
        ) }}
        {{ tag('img', { src: possibleAsset.url }) }}
        {{ possibleAsset.getImg({ width: 100, height: 100 }) }}
        {{ possibleAsset.filename }}
      </label>
    </li>
  {% endfor %}
</ul>
```

You could then make the checkbox list sortable, so users have control over the order of related assets.

#### Creating New Assets

Assets fields can handle new file uploads as well:

```twig
{{ input('file', 'fields[myFieldHandle][]', options={
  multiple: true,
}) }}
```

::: tip
Don’t forget to set `enctype="multipart/form-data"` on your `<form>` tag so your browser knows to submit the form as a multipart request.
:::

Alternatively, you can submit Base64-encoded file data, which the Assets field will decode and treat as an uploaded file. To do that, you have to specify both the data and the filename like this:

```twig
{{ hiddenInput(
  'fields[myFieldHandle][data][]',
  'data:image/jpeg;base64,my-base64-data'
) }}
{{ hiddenInput('fields[myFieldHandle][filename][]', 'myFile.ext') }}
```

However you upload new assets, you may want to maintain any that already exist on the field and append new ones to the set instead of replacing it.

You can do this by passing each of the related asset IDs in the field data array, like we are here with hidden form inputs:

```twig{1-8}
{# Provide each existing asset ID in the array of field data #}
{% for relatedAssetId in entry.myFieldHandle.ids() %}
  {{ input(
    'hidden',
    'fields[myFieldHandle][]',
    relatedAssetId
  ) }}
{% endfor %}

{{ input('file', 'fields[myFieldHandle][]', options={
  multiple: true,
}) }}
```

## See Also

- [Asset Queries](assets.md#querying-assets)
- <craft3:craft\elements\Asset>
- [Relations](relations.md)
