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
