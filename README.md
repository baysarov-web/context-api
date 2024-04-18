# Контекст

> **Примечание**
> 
> Данный материал – корректировка [главы](https://ru.reactjs.org/docs/context.html) из документации по React. Из корректировок: примеры переведены на функциональные компоненты. Также убраны некоторые абзацы, которые сейчас не актуальны.

**Контекст позволяет передавать данные через дерево компонентов без необходимости передавать пропсы на промежуточных уровнях.**

В типичном React-приложении данные передаются сверху вниз (от родителя к дочернему компоненту) с помощью пропсов. Однако, подобный способ использования может быть чересчур громоздким для некоторых типов пропсов (например, выбранный язык, UI-тема), которые необходимо передавать во многие компоненты в приложении. Контекст предоставляет способ делиться такими данными между компонентами без необходимости явно передавать пропсы через каждый уровень дерева.

## Когда использовать контекст

Контекст разработан для передачи данных, которые можно назвать «глобальными» для всего дерева React-компонентов (например, текущий аутентифицированный пользователь, UI-тема или выбранный язык). В примере ниже мы вручную передаём проп `theme`, чтобы стилизовать компонент `Button`:

```javascript
function App() {
  return <Toolbar theme="dark" />;
}

function Toolbar (props) {
  // Компонент Toolbar должен передать проп "theme" ниже,
  // фактически не используя его. Учитывая, что у вас в приложении
  // могут быть десятки компонентов, использующих UI-тему,
  // вам придётся передавать проп "theme" через все компоненты.
  // И в какой-то момент это станет большой проблемой.
  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

function ThemedButton (props) {
  return <Button theme={props.theme} />;
}
```

Контекст позволяет избежать передачи пропсов в промежуточные компоненты:

```javascript
// Контекст позволяет передавать значение глубоко
// в дерево компонентов без явной передачи пропсов
// на каждом уровне. Создадим контекст для текущей
// UI-темы (со значением "light" по умолчанию).
const ThemeContext = React.createContext('light');

function App () {
  // Компонент Provider используется для передачи текущей
  // UI-темы вниз по дереву. Любой компонент может использовать
  // этот контекст и не важно, как глубоко он находится.
  // В этом примере мы передаём "dark" в качестве значения контекста.
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

// Компонент, который находится в середине,
// больше не должен явно передавать тему вниз.
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton () {
  // Используем useContext(), чтобы получить значение контекста.
  // React найдёт (выше по дереву) ближайший Provider-компонент,
  // предоставляющий этот контекст, и использует его значение.
  // В этом примере значение UI-темы будет "dark".
  const context = useContext(ThemeContext);
  
  return <Button theme={context} />;
}
```

## API

### `React.createContext`

```js
const MyContext = React.createContext(defaultValue);
```

Создаёт объект `Context`. Когда React рендерит компонент, который подписан на этот объект, React получит текущее значение контекста из ближайшего подходящего `Provider` выше в дереве компонентов.

Аргумент `defaultValue` используется **только** в том случае, если для компонента нет подходящего `Provider` выше в дереве. Значение по умолчанию может быть полезно для тестирования компонентов в изоляции без необходимости оборачивать их. Обратите внимание: если передать `undefined` как значение `Provider`, компоненты, использующие этот контекст, не будут использовать `defaultValue`.

### `Context.Provider`

```js
<MyContext.Provider value={/* некоторое значение */}>
```

Каждый объект `Context` используется вместе с `Provider` компонентом, который позволяет дочерним компонентам, использующим этот контекст, подписаться на его изменения.

Компонент `Provider` принимает проп `value`, который будет передан во все компоненты, использующие этот контекст и являющиеся потомками этого компонента `Provider`. Один `Provider` может быть связан с несколькими компонентами, потребляющими контекст. Так же компоненты `Provider` могут быть вложены друг в друга, переопределяя значение контекста глубже в дереве.

Все потребители, которые являются потомками `Provider`, будут повторно рендериться, как только проп `value` у `Provider` изменится. 
Изменения определяются с помощью сравнения нового и старого значения, используя алгоритм, аналогичный [`Object.is`](//developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description).

## Примеры

### Динамический контекст

Более сложный пример динамических значений для UI темы:

**theme-context.js**
```javascript
import React from 'react'

export const themes = {
  light: {
    foreground: '#000000',
    background: '#eeeeee',
  },
  dark: {
    foreground: '#ffffff',
    background: '#222222',
  },
};

export const ThemeContext = React.createContext(
  themes.dark // значение по умолчанию
);
```
**ThemedButton.js**
```javascript
import { ThemeContext } from './theme-context';
import { useContext } from 'react';

function ThemedButton (props) {
  const theme = useContext(ThemeContext);

  return (
    <button
      {...props} // прокидываем все принятые пропсы в button
      style={{backgroundColor: theme.background}}
    >
      click
    </button>
  );
}

export default ThemedButton
```

**App.js**

```javascript
import { ThemeContext, themes } from './theme-context';
import ThemedButton from './ThemedButton';
import { useState } from 'react'

// Промежуточный компонент, который использует ThemedButton
function Toolbar(props) {
  return (
    <ThemedButton onClick={props.changeTheme}>
      Change Theme
    </ThemedButton>
  );
}

function App () {
  const [theme, setTheme] = useState(themes.light);

  const toggleTheme = () => {
    console.log()
    setTheme(theme === themes.dark ? themes.light : themes.dark);
  }

  // ThemedButton внутри ThemeProvider использует
  // значение светлой UI-темы из состояния, в то время как
  // ThemedButton, который находится вне ThemeProvider,
  // использует тёмную UI-тему из значения по умолчанию
  return (
    <div>
      <ThemeContext.Provider value={theme}>
        <Toolbar changeTheme={toggleTheme} />
      </ThemeContext.Provider>
      <div>
        <ThemedButton />
      </div>
    </div>
  );
}

export default App;
```

### Изменение контекста из вложенного компонента

Довольно часто необходимо изменить контекст из компонента, который находится где-то глубоко в дереве компонентов. В этом случае вы можете добавить в контекст функцию, которая позволит потребителям изменить значение этого контекста:

**theme-context.js**
```javascript
// ... 
export const ThemeContext = React.createContext({
  theme: themes.dark,
  toggleTheme: () => {},
});
```

**ThemeTogglerButton.js**
```javascript
import { ThemeContext } from './theme-context';
import { useContext } from 'react'

function ThemeTogglerButton() {
  // ThemeTogglerButton получает из контекста
  // не только значение UI-темы, но и функцию toggleTheme.
  const context = useContext(ThemeContext);

  return (
    <div>
      <button
        onClick={context.toggleTheme}
        style={{backgroundColor: context.theme.background}}>
        Toggle Theme
      </button>
    </div>
  );
}

export default ThemeTogglerButton;
```

**App.js**

```javascript
import { ThemeContext, themes } from './theme-context';
import ThemeTogglerButton from './ThemeTogglerButton';
import { useState } from 'react'

function App () {
  const [theme, setTheme] = useState(themes.light);

  const toggleTheme = () => {
    if(theme === themes.light) {
      setTheme(themes.dark);
    } else {
      setTheme(themes.light);
    }
  }

  // В качестве значения контекста передаётся само значение и функция
  // для изменения этого значения
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <Content />
    </ThemeContext.Provider>
  );

}

// Вспомогательный компонент для демонстрации вложенности
function Content() {
  return (
    <div>
      <ThemeTogglerButton />
    </div>
  );
}

export default App;
```

## Дополнительный материал

– [React Context за 5 минут: что это и как использовать](https://tproger.ru/translations/react-context-in-5-min/) <sup>классы</sup>  
– [Как эффективно применять React Context](https://habr.com/ru/post/522896/)  
– [Understand React Context API](https://www.telerik.com/blogs/understand-react-context-api) <sup>en</sup>  
– [Guide to React Context](https://ui.dev/react-context/) <sup>en</sup>  
