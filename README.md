# Astro getTaxonomy Slug auto(EN/RU)

```ts
import { getCollection } from "astro:content";
import { slug } from "github-slugger";
import slugify from "slugify";

export const customSlugify = (content) => {
  if (!content) return null;

  // Проверка, содержит ли строка русские символы
  const isRussian = /[а-яА-ЯЁё]/.test(content);

  if (isRussian) {
    // Используем slugify для русских символов
    return slugify(content, { lower: true, strict: true, locale: "ru" });
  } else {
    // Используем стандартный slug для английских символов
    return slug(content);
  }
};

// Функция для преобразования строки в slug
// export const slugify = (content) => {
//   if (!content) return null;
//   return slug(content);
// };

// Функция для получения страниц, исключая черновики и страницы с определенными ID
export const getSinglePage = async (collection) => {
  const allPage = await getCollection(collection);
  const removeIndex = allPage.filter((data) => data.id.match(/^(?!-)/));
  const removeDrafts = removeIndex.filter((data) => !data.data.draft);
  return removeDrafts;
};

// Функция для получения всех таксономий
export const getTaxonomy = async (collection, name) => {
  const singlePages = await getSinglePage(collection);
  const taxonomyPages = singlePages.map((page) => page.data[name]);
  let taxonomies = [];
  for (let i = 0; i < taxonomyPages.length; i++) {
    const categoryArray = taxonomyPages[i];
    for (let j = 0; j < categoryArray.length; j++) {
      // customSlugify <-> slugify
      taxonomies.push(customSlugify(categoryArray[j])); // Используйте customSlugify здесь
    }
  }
  const taxonomy = [...new Set(taxonomies)];
  return taxonomy;
};

```

# markdownify

Преобразует Markdown-форматированный текст в HTML. Это полезно, когда нужно отобразить текст, написанный в Markdown, в виде HTML на веб-странице.

```ts
export const markdownify = (content: string) => {
  if (!content) return null;

  return marked.parseInline(content);
};
```
## Пример
```js
const markdownText = "# Заголовок\nЭто *курсив* и это **жирный** текст.";
const htmlContent = markdownify(markdownText);
// Результат: "<h1>Заголовок</h1><p>Это <em>курсив</em> и это <strong>жирный</strong> текст.</p>"
```

# humanize

Преобразует строку, содержащую подчеркивания или пробелы, в читаемый вид, делая первую букву заглавной. Это полезно для отображения идентификаторов или кодовых имен в более понятной форме.

```ts
// humanize
export const humanize = (content: string) => {
  if (!content) return null;

  return content
    .replace(/^[\s_]+|[\s_]+$/g, "")
    .replace(/[_\s]+/g, " ")
    .replace(/^[a-z]/, function (m) {
      return m.toUpperCase();
    });
};
```
## Пример
```js
const codeName = "hello_world_example";
const humanizedName = humanize(codeName);
// Результат: "Hello world example"
```

# plainify & htmlEntityDecoder

plainify & htmlEntityDecoder: Удаляют HTML-теги и декодируют HTML-сущности, превращая HTML-контент в обычный текст. Это полезно, если вам нужно извлечь чистый текст из HTML, например, для отображения в текстовом редакторе или для дальнейшей обработки.

```ts
// plainify
export const plainify = (content: string) => {
  if (!content) return null;

  const filterBrackets = content.replace(/<\/?[^>]+(>|$)/gm, "");
  const filterSpaces = filterBrackets.replace(/[\r\n]\s*[\r\n]/gm, "");
  const stripHTML = htmlEntityDecoder(filterSpaces);
  return stripHTML;
};

// strip entities for plainify
const htmlEntityDecoder = (htmlWithEntities: string): string => {
  let entityList: { [key: string]: string } = {
    "&nbsp;": " ",
    "&lt;": "<",
    "&gt;": ">",
    "&amp;": "&",
    "&quot;": '"',
    "&#39;": "'",
  };
  let htmlWithoutEntities: string = htmlWithEntities.replace(
    /(&amp;|&lt;|&gt;|&quot;|&#39;)/g,
    (entity: string): string => {
      return entityList[entity];
    }
  );
  return htmlWithoutEntities;
};
```
## Пример
```js
const htmlContent = "<p>Hello <b>World</b> &amp; everyone!</p>";
const plainText = plainify(htmlContent);
// Результат: "Hello World & everyone!"
```
