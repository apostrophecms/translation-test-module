This bundle allows automatic translation of documents (pages and pieces) when localizing content. It comes with two translation providers: Google Cloud Translation and DeepL.

It is also possible to configure your own providers, [see related section](#custom-translation-providers).

## Installation

To install the module, use the command line to run this command in an Apostrophe project's root directory:

```sh
npm install @apostrophecms-pro/automatic-translation
```

## Setup

Configure the modules in your `app.js` file, depending on the provider you want to use.

### DeepL Setup

First, you need to get an API authentication key from your [DeepL Account](https://www.deepl.com/your-account/summary).
Copy the key from the "Authentication Key for DeepL API" section and add it as an environment variable.

It is recommended to use environment variables or a configuration file to store the key.
The DeepL module will recognize and automatically use the environment variable `APOS_DEEPL_API_SECRET`:

```bash
export APOS_DEEPL_API_SECRET=your-key-here

npm start
```

When using the environment variable, your configuration will look like this:

```javascript
require('apostrophe')({
  shortName: 'my-project',
  modules: {
    '@apostrophecms-pro/automatic-translation': {
      options: {
        provider: 'deepl'
      }
    },
    '@apostrophecms-pro/automatic-translation-deepl': {}
  }
});
```

It's not a good practice to store secrets in your code but they can be passed to your module options directly:

```javascript
require('apostrophe')({
  shortName: 'my-project',
  modules: {
    '@apostrophecms-pro/automatic-translation': {
      options: {
        provider: 'deepl'
      }
    },
    '@apostrophecms-pro/automatic-translation-deepl': {
      options: {
        apiSecret: 'your-key-here'
      }
    }
  }
});
```

:warning: Your project locales might not match exactly source and target languages from `Deepl`.
You can set mappings by adding in the module options `sourcesMapping` and `targetsMapping` to map your project locales to Deepl supported languages.

The keys of the mappings correspond to your project locales and the values to the Deepl supported source and target languages.

Here is how to configure mapping for sources, in this case `en-US` is your project locale while `en` is the Deepl supported language.
Deepl sources only support languages without country codes.

That's why you might not need this mapping since we remove the country code from your locales by default (if existing).
```javascript
'@apostrophecms-pro/automatic-translation-deepl': {
  options: { 
    sourcesMapping: {
      'en-US': 'en',    
    },
  }
}
```

Here is how to configure mapping for targets. 
In this case `en-GB` is your project locale while `en-US` is the Deepl one you want to translate to.

```javascript
'@apostrophecms-pro/automatic-translation-deepl': {
  options: { 
    // These are the default values that you can override
    targetsMapping: {
      'en-GB': 'en-US', // For targets, `en` is not supported
    }
  }
}
```

Note that some default values have been set for `targetsMapping` to avoid you seeing errors for `en` and `pt` locales 
(these locales not being supported by Deepl for targets).
You can override them as needed.

```javascript
  targetsMapping: {
    en: 'en-US',
    pt: 'pt-PT'
  }
```

For a better understanding of the supported languages, 
you can check the [Deepl documentation](https://developers.deepl.com/docs/resources/supported-languages).

### Google Cloud Translation Setup

First, you need to create a project in the [Google Cloud Console](https://cloud.google.com/) or use an existing one, and enable the Cloud Translation API for the project. Note the Project ID of your project, you'll need that later. You can find more information in the [Google Cloud Translation documentation](https://cloud.google.com/translate/docs/setup).

As a final step, you need to create a service account and download the JSON key file in order to authenticate your requests.
You can follow the steps for creating a service account key in the [Google Cloud IAM documentation](https://cloud.google.com/iam/docs/keys-create-delete).
As a result of that step, you should have downloaded a JSON file with your service account key.

It is preferable to use environment variables for sensitive information, you can export the following before starting your application:

```bash
export APOS_GOOGLE_TRANSLATE_PROJECT_ID=your-project-id
export APOS_GOOGLE_TRANSLATE_KEY_FILENAME=/path/to/your/keyfile.json

npm start
```

Configure the module in the `app.js` file:

```javascript
require('apostrophe')({
  shortName: 'my-project',
  modules: {
    '@apostrophecms-pro/automatic-translation': {
      options: {
        provider: 'google'
      }
    },
    '@apostrophecms-pro/automatic-translation-google': {}
  }
});
```

It's also possible to pass sensitive information in your module options directly, even if not recommended:

```javascript
require('apostrophe')({
  shortName: 'my-project',
  modules: {
    '@apostrophecms-pro/automatic-translation': {
      options: {
        provider: 'google'
      }
    },
    '@apostrophecms-pro/automatic-translation-google': {
      options: {
        projectId: 'your-project-id',
        keyFilename: '/path/to/your/keyfile.json'
      }
    }
  }
});
```

**That's it! You're good to go!** Below you can find more information about how to fine-tune your setup if needed.

## Usage

When configured, the module will automatically translate the content of your documents when you localize them.
It will show a new `Translate text content` option under the `Automatic translation settings` section in the localization dialog.

![Localization Wizard](https://static.apostrophecms.com/apostrophecms/automatic-translation/images/localization-wizard.png)

The module will properly handle the supported languages of the current translation provider.
It will skip the translation when a language is not supported, and show a message about it.

Only languages that are both present in your configured locales and supported by the translation provider will be offered.

All text fields (type `string` and `slug`), rich text widgets and generally all widgets that contain text-based schema fields will be translated. The `html` widget is disabled by default. You can enable it in your project by setting the `translate` option to `true` in the widget module definition (see the "Configure widgets" section below).

### Configure fields

The module will translate `string` and `slug` fields by default, no matter how deep they are in the document. If you want to opt-out some fields from the translation, you can use the `translate` option in the field definition:

```javascript
// in modules/my-module/index.js
module.exports = {
  fields: {
    add: {
      myField: {
        type: 'string',
        label: 'My Field',
        translate: false
      }
    }
  }
};
```

### Configure widgets

By default, all widgets will be translated. The module will attempt to translate the schema fields of any widget that is found in the document. The `rich-text` widget is also handled properly. If you want to opt-out all fields of a particular widget type, you can use the `translate` option of the widget module:

```javascript
// in modules/my-widget/index.js
module.exports = {
  options: {
    label: 'My Widget',
    translate: false
  }
};
```

Or you can opt-out an instance of a widget when adding it to an area field:

```javascript
// in modules/my-module/index.js
module.exports = {
  fields: {
    add: {
      myArea: {
        type: 'area',
        label: 'My Area',
        options: {
          widgets: {
            'my-widget': {
              options: {
                translate: false
              }
            }
          }
        }
      }
    }
  }
};
```

## Custom field types

If you have custom field types that contain text, and you want to translate them, you can adapt the type slightly to support that. Here is a basic example of a custom field type that supports translation:

```javascript
// in modules/my-module/index.js
module.exports = {
  init(self) {
    self.apos.schema.addFieldType({
      // This indicates that the field type supports translation
      translate: true,
      name: 'myCustomField',
      // Let's assume the field contains just text
      convert(req, field, data, object) {
        const input = data[field.name];
        object[field.name] = self.apos.launder.string(input);
      },
      vueComponent: 'MyCustomField'
    });
  }
};
```

In this example, we have a simple field type that contains plain text (not HTML). We set the `translate` option to `true` and the module will automatically translate the field when localizing the document.

A more advanced example would be a field type that contains e.g., an object with some text properties. In that case, you would need to implement the `getText` and `convertText` methods in the field type definition:

```javascript
// in modules/my-module/index.js
module.exports = {
  init(self) {
    self.apos.schema.addFieldType({
      // This indicates that the field type supports translation
      translate: true,
      name: 'myCustomField',
      convert(req, field, data, object) {
        // ... your logic here
      },
      getText(field, value, valuePath) {
        // let's assume the field value contains two properties of interest -
        // `text1` and `text2`.
        // For every item of the returned array, `convertText` method will be called
        // with `translated` value and the item as `meta` object.
        return [
          // Export `text1` for translation
          {
            valuePath: [ ...valuePath, field.name, 'text1' ],
            text: value.text1,
            type: field.type
          },
          // Export `text2` for translation
          {
            valuePath: [ ...valuePath, field.name, 'text2' ],
            text: value.text2,
            type: field.type
          },
          // You can also export only meta information in order to
          // mark the field as "translated" in the UI. Add custom property
          // `original` that can be accessed in the `convertText` method.
          // See full explanation of metaOnly below.
          {
            metaOnly: true,
            valuePath: [ ...valuePath, field.name ],
            original: value,
            type: field.type
          }
        ]
      },
      // `translated` is our entire object for the meta only field, and the
      // text for the other fields that we have exported in the `getText` method.
      // `meta` is the meta information that we have exported in the `getText` method.
      convertText(translated, meta) {
        // If the metaOnly result is passed, we can compute.
        // See full explanation of metaOnly below
        if (meta.metaOnly) {
          const { original, valuePath, type } = meta.original;
          // Compute the changed state of the field.
          // This data will be available in the UI indicator component
          return {
            valuePath,
            type,
            changed: original.text1 !== translated.text1 || original.text2 !== translated.text2
          };
        }

        // For our text data, we can use the `he` library to decode HTML entities
        // that were eventually encoded by the translation provider.
        // Or you can just return the translated text as is.
        const converted = he.decode(translated || '');
        return {
          ...meta,
          translated: converted,
          changed: meta.text.trim() !== converted.trim()
        };
      },
      vueComponent: 'MyCustomField'
    });
  }
};
```

Learn more about the translation and field metadata in the dedicated section below.

In your Vue component, you can pass the meta information so that the "Translated" indicator can be displayed:

```vue
<template>
  <!-- Add :meta="fieldMeta" prop here -->
  <AposInputWrapper
    :modifiers="modifiers" :field="field"
    :error="effectiveError" :uid="uid"
    :display-options="displayOptions"
    :meta="fieldMeta"
  >
  <!-- ... your input component here -->
  </AposInputWrapper>
</template>

<script>
import AposInputMixin from 'Modules/@apostrophecms/schema/mixins/AposInputMixin';
export default {
  name: 'AposInputString',
  // the mixin will handle the meta information
  mixins: [ AposInputMixin ],
  // ... your component logic here
}
</script>
```

## Advanced widget support

Your widget can override the default text extraction if needed. By default, the widgets will use `getText` and `convertText` on their underlying field schema. If this is not working for your custom widget, you can override this behavior and return your desired list of properties to be translated. You can do this by implementing the `getTranslationText` and `convertTranslationText` methods in your widget module:

```javascript
// in modules/my-widget/index.js
module.exports = {
  extend: '@apostrophecms/widget',
  options: {
    label: 'My Widget',
    translate: true
  },
  methods(self) {
    return {
      getTranslationText(value) {
        // ... your logic here
      },
      convertTranslationText(translated, meta) {
        // ... your logic here
      }
    };
  }
};
```

`getTranslationText` and `convertTranslationText` methods implement the same logic as `getText` and `convertText` methods in the custom field type example. The only difference is that `getTranslationText` receives only the widget value. The method names are different to avoid conflicts with existing methods in the widget modules. Learn more about the translation meta and the methods logic in the next section.

Your custom translation methods should carry the logic of extracting and converting the text from the widget value.
You can look at the code in `modules/@apostrophecms-pro/automatic-translation-rich-text-widget` for an advanced example of how to handle widget-specific text extraction and conversion.


## Translation meta and field meta explained

First, let's clarify the terms.

**Translation meta**, in the context of this module, is the information extracted from a document in the form of a list of objects, each containing information about:
- the path to a text value in the document
- the type of field that owns the text
- any additional information that might be useful for the translation provider, the current field type or the UI

Keep in mind that one schema field can have multiple text values, and each of them will be translated separately, and exported as a separate meta object.

**Field meta** is field-specific information that is added to the document. This is a core feature that allows the storage of additional information about a schema field. This feature is currently used to store the "changed" status of the field after translation, the "original" text and the document value path to the text.

**How it works?**
1. The document translation meta is extracted from the document. It contains per-field meta (as explained above). Here is where the `getText` method per field is used.
2. The translation provider translates the text and returns the translated text.
3. The `convertText` method is called on every translated text. This allows the field type to perform any formatting on the translated text (e.g. escape HTML entities when required) but also to add additional meta information. For example, we add `changed` status by comparing the original and translated text.
4. The translated text replaces the original text in the document.
5. The translation meta per field is stored as a field meta in the document.
6. The UI can use the field meta to display the "Translated" indicator.


### Basic example

Let's look closer at the `getText` logic and its return format. A typical text field will be handled similarly to this:

```js
getText(field, value, valuePath) {
  return [
    {
      valuePath: [ ...valuePath, field.name ],
      text: value,
      type: field.type
    }
  ];
}
```

The `field` argument is the schema field definition, `value` is the value of the field in the document. `valuePath` is an array of parent field names that lead to the current field, something we call `pathComponents`. The method should always (with one exception explained below) return an array of objects, each containing the `valuePath`, the `text` and the `type` of the field. Adding more arbitrary properties to the object is allowed.

With the simple example above, the next step according to our logic is to call the `convertText` method for each of the returned objects when the translation is done. The `convertText` method will receive the translated text and the meta object. Here is an example of how the corresponding `convertText` method can look like:

```js
convertText(translated, meta) {
  const converted = he.decode(translated || '');

  return {
    ...meta,
    translated: converted,
    changed: meta.text.trim() !== converted.trim()
  };
}
```

The same `meta` object that was returned from the `getText` method is passed to the `convertText` method. We add the `translated` property to the meta object after converting the translated text.

We use the `he` package to unescape any HTML entities that might have been encoded by the translation provider. We also add the `changed` status to the meta object. This status is used by the UI to display the "Translated" indicator. Any additional properties that are added to the meta object will be stored later as field meta in the document.

After the conversion, our meta object will look like this:

```js
{
  valuePath: [ 'myField' ],
  text: 'Original text',
  type: 'string',
  translated: 'Translated text',
  changed: true
}
```

The internal bundle engine will first use this to replace the translated text in the document. It will set `doc.myField` to `Translated text`. Then, it will store some of the properties as field meta in the document, using code similar to the following:

```js
self.apos.schema.setMeta(doc, '@apostrophecms/automatic-translation', 'myField', 'data', {
  valuePath: 'myField',
  text: 'Original text',
  type: 'string',
  changed: true
});

```

The payload (last argument) will be stored in a key, `data` for the path (path components) `myField` under namespace `@apostrophecms/automatic-translation` in the document.

### Private meta properties

It's possible to pass useful data between `getText` and `convertText` while keeping those private, not saved as field meta in the document. Here is an example with a private property `original`:

```js
getText(field, value, valuePath) {
  // Logic that would extract `/parent-path/my-slug` from the value
  // and transform it to `my slug`.
  const slugText = transformSlugToText(value);
  return [
    {
      valuePath: [ ...valuePath, field.name ],
      text: slugText,
      original: value,
      type: field.type,
    }
  ];
},
convertText(translated, meta) {
  const { original, ...rest } = meta;
  // Logic that will transform `my translated slug` 
  // to `/parent-path/my-translated-slug`
  const slug = transformTextToSlug(translated, original);

  // Omit the `original` property from the meta object
  return {
    ...rest,
    translated: slug,
    changed: original.trim() !== slug.trim()
  };
}
```

The `original` property is introduced in the `getText` method and is passed to the `convertText` method after successful translation. The `original` property is omitted from the returned meta object in the `convertText` method. The `original` property is not saved as metadata in the document. It's a private property used internally to deliver data between the `getText` (before translation) and `convertText` (after translation) methods.

### Unique value path

The previous example works well for simple, root-level document properties. What about widgets and array items? The problem with these is that their position in the document may change. To address this, we need to use a unique value path. Luckily, this is already handled well by the Apostrophe core. Every array or area `item` has a unique `_id` property. A unique path to any such item is possible by using `@{_id}` as a starting path component. Here is an example of a rich text widget value:

```js
{
  _id: 'uniqueId',
  content: 'Original text',
  // ... other properties
}
```

Its path components would be:

```js
[ '@uniqueId', 'content' ]
```

The string version of the path above is `@uniqueId.content`. The methods for the rich text widget would look like this:

```js
getTranslationText(value) {
  return [
    {
      valuePath: [ `@${value._id}`, 'content' ],
      metaPath: [ `@${value._id}` ],
      text: value.content,
      type: field.type
    }
  ];
},
convertTranslationText(translated, meta) {
  // No need to transform the text, as it's already HTML
  return {
    ...meta,
    translated,
    changed: meta.text.trim() !== translated.trim()
  };
}
```

You might have noticed that we have introduced a new `metaPath` property. Most of the time the path components passed to the `apos.schema.setMeta()` are the same as our `valuePath` that represents the path components to the document value. However, in the case of more complex value structures, this won't be the case anymore. When available, `metaPath` will be used to store the field meta in the document:

```js
self.apos.schema.setMeta(doc, '@apostrophecms/automatic-translation', '@uniqueId', 'data', {
  valuePath: '@uniqueId.content',
  text: 'Original text',
  type: 'string',
  changed: true
});
```

Here we have `widgetValue.content` property, containing the text we want to translate. The `content` property is an internal implementation detail and it should not be used when adding the field meta. We can use `metaPath` in order to provide the value as required by the `apos.schema.setMeta()` method.

### Meta only fields and the `metaOnly` property

Let's look at another scenario - array fields. The array field does not directly participate in translation but returns the array of fields that should be translated for its schema. At the same time, we need a way to add field meta for the array field so that the UI can see that this array field has "changed" status. We can use a separate meta object and the `metaOnly` property for that. Here is an example of how the `getText` and `convertText` methods can look like for an array field (pseudo code):

```js
getText(field, value, valuePath) {
  // Extract the translation meta for the array items
  const meta = getChildrenMeta(field.schema, value);

  // Add the meta for the array field itself
  if (meta.length) {
    meta.push({
      metaOnly: true,
      valuePath: [ ...valuePath, field.name ],
      type: field.type
    });
  }

  return meta;
},
convertText(translated, meta) {
  // This should never happen.
  if (!meta.metaOnly) {
    throw new Error('This method should be called only for my meta only fields');
  }
  return {
    ...meta,
    changed: translated.length > 0
  };
}
```
The `getText` retrieves the translation meta for the array items. It also adds a meta object for the array field itself. It sets the `metaOnly` property to `true` in the array field meta. The `text` property is ignored. This meta object will NOT be sent to the translation provider. It will be used to store field meta in the document after the translation is performed and added to the document. Furthermore, the `convertText` method will be called only for the meta `array` field. The `translated` argument will be the array value retrieved from the document, using the `valuePath` provided in the array meta object. The engine uses `apos.util.get(valuePath)` under the hood to retrieve the value from the document. The `convertText` method will add the `changed` status to the meta object and the engine will store the meta object as field meta in the document:

```js
self.apos.schema.setMeta(doc, '@apostrophecms/automatic-translation', 'myArray', 'data', {
  valuePath: 'myArray',
  type: 'array',
  changed: true
});
```

## Custom Translation Providers

Creating your own translation provider is also supported.

1. Create a folder `modules/my-provider` in your project. Add a file `index.js` with the following content:

```javascript
module.exports = {
  init(self, options) {
    self.apos.modules['@apostrophecms-pro/automatic-translation']
      .registerProvider(self, {
        name: 'my-provider',
        label: 'My Provider'
      });
  },

  methods(self) {
    return {
      async translate(req, data, sourceLanguage, targetLanguage, options) {
        // Get the text to translate from the `source` language code
        // to the `target` language code
        const texts = data.fields.map(m => m.text);

        // Your translation logic here. Array of translated text (string) is expected
        // with the exact same order (index) as the input text.
        // Your provider should support HTML text. The internal engine expects HTML support and
        // unescapes HTML entities for non-HTML text fields after successful translation.
        const translatedTextArray = self.translateTextWithMyProvider(
          texts,
          sourceLanguage,
          targetLanguage
        );

        // Finally, return the translated text in this format.
        // The `state` can be `translated` or `failed`.
        // If it's `failed`, fields should be empty. It's a good practice to offer
        // a structured logs (self.logError(...)) for the failed translations.
        return {
          state: 'translated',
          fields: translatedTextArray
        };
      },

      async getSupportedLanguages(req, sourceLanguages, targetLanguages) {
        // `sourceLanguages` and `targetLanguages` are optional arrays of language codes.
        // If they are provided, the method should return information about whether
        // each language is supported as a source or a target respectively.
        // if a requested language is not supported, then the supported flag will be false.

        // The expected return format is:
        return {
          source: [
            { code: 'en', supported: true },
            { code: 'fr', supported: true },
            { code: 'zh', supported: false }
          ],
          target: [
            { code: 'en', supported: true },
            { code: 'fr', supported: true },
            { code: 'zh', supported: true }
          ]
        };
      }
    };
  }
};
```

2. Add your logic to the code above and configure the module in your `app.js` file:

```javascript
require('apostrophe')({
  shortName: 'my-project',
  modules: {
    '@apostrophecms-pro/automatic-translation': {
      options: {
        provider: 'my-provider'
      }
    },
    'my-provider': {}
  }
});
```
