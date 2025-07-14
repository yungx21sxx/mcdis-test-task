# mcdis-test-task

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```

# Button.vue

Компонент универсальной кнопки с разными визуальными состояниями.

## Props

- `variant` — определяет стиль и поведение кнопки.  
  Поддерживаемые значения:
    - `'DEFAULT'` — обычная активная кнопка
    - `'DISABLED'` — неактивная (задизейблена)
    - `'FOCUSED'` — активная с фокус-стилем

## Стили

В зависимости от `variant` компонент добавляет соответствующий CSS-класс:

- `btn_default` — при `variant === 'DEFAULT'`
- `btn_disabled` — при `variant === 'DISABLED'`
- `btn_focused` — при `variant === 'FOCUSED'`

Также всегда добавляется базовый класс `btn`.

## Поведение

- Кнопка становится неактивной (`disabled`), если передан `variant === 'DISABLED'`.
- В остальных случаях работает как обычная кнопка с типом `button`.

## Слот

Внутрь кнопки можно передавать любой контент через слот. Например:

# FileInput.vue

Компонент для выбора и загрузки одного файла. Поддерживает смену состояния (выбран, загружается, загружен), отображает ошибки и управляет кнопкой действия.

---

## Props

- `label` (string, по умолчанию `'Label'`) — подпись к полю выбора файла.
- `hint` (string, по умолчанию `'Hint text'`) — дополнительный текст-подсказка под инпутом.
- `error` (string или null) — текст ошибки, отображается при наличии.
- `required` (boolean, по умолчанию `false`) — делает поле обязательным.
- `disabled` (boolean, по умолчанию `false`) — отключает выбор файла и кнопку.
- `id` (string, по умолчанию пустая строка) — кастомный ID инпута.
- `uploading` (boolean, по умолчанию `false`) — если true, отображается состояние загрузки (и кнопка "Отменить").

---

## Emits

- `update:files` — эмитит выбранные файлы в виде массива `File[]`.
- `error` — отправляет сообщение об ошибке валидации или загрузки.
- `cancel` — сигнал на отмену загрузки.
- `delete` — передаёт файл, который был удалён.
- `update:uploading` — обновляет состояние загрузки (true/false).

---

## Состояния компонента

Компонент может находиться в одном из четырёх состояний:

1. `empty` — файл не выбран.
2. `selected` — файл выбран, но загрузка ещё не началась.
3. `uploading` — активна загрузка.
4. `uploaded` — файл успешно загружен.

Состояние определяется автоматически по выбранным файлам и флагу `uploading`.

---

## Логика поведения

- При клике на кнопку в `empty` или `selected` состоянии открывается диалог выбора файла.
- В `uploading` состоянии кнопка меняется на "Отменить загрузку" и эмитит `cancel`.
- В `uploaded` состоянии кнопка показывает "Удалить", и при нажатии файл удаляется.
- Ошибки можно передавать через проп `error` или установить локально через `internalError`.

---



