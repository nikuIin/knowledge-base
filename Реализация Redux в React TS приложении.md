[[Frontend.canvas|Frontend canvas]]
[[Redux]]
[Туториал по Redux](https://www.youtube.com/watch?v=5yEG6GhoJBs&t=2080s
[Код из видео](https://github.com/cosdensolutions/code/tree/master/videos/long/redux-tutorial)

 Как мы уже обозначали, в **Redux** есть несколько основных понятий:
- `Store` — единое место для хранения **глобальной информации** и **глобальных данных**.
- `Redux` — это место, где мы описываем **чистые функции**.
  - **Чистая функция** — функция, которая всегда имеет одно и то же поведение при одинаковых входных данных и не имеет побочных эффектов (не влияет на приложение).
  - В этих чистых функциях мы описываем **определённое действие**.

- `Actions` (экшены) — связываются с чистыми функциями в Redux.  
  - Экшены должны через своё название **прямо передавать и описывать**, что должно произойти.  
  - Примеры названий экшенов:  
    - `SetUser`  
    - `ClearUser`  
    - `CounterIncrement`  
    - `CounterDecrement`  
    - `CounterIncrementByValue`

- `Dispatch` — принимает какой-либо экшен и **отправляет его в Redux**.  
  - Технически Dispatch не отправляет экшен напрямую в `Store`, а в Redux.  
  - Redux уже выбирает, какую **чистую функцию вызвать**, которая изменит состояние `Store`.


1) Создадим сначала `slice` — `UserSlice`  и опишем там `action` и `reducer`:

```tsx
// UserSlice.tsx
import type { User } from "@entities/user";
import { type PayloadAction, createSlice } from "@reduxjs/toolkit";

interface UserState {
  user: User | null;
}

const initialState: UserState = { user: null };

const userSlice = createSlice({
  name: "user",
  initialState: initialState,
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
    },
    clearUser: (state) => {
      state.user = null;
    },
  },
});

export const { setUser, clearUser } = userSlice.actions;
export const userReducer = userSlice.reducer;
```

Благодаря `TS` мы можем легко и быстро создавать `reducers` и `actions`, а именно в одном `createSlice()`.

В ключе `reducers` каждый элемент — это `action`. Потом мы просто обращаемся к `userSlice.actions` для их экспортирования. Вот как все просто.
Та же ситуация с `reducer`.

2) Определяем `store`

```tsx
// Store.tsx
import { combineReducers, configureStore } from "@reduxjs/toolkit";
import { userReducer } from "@entities/user/model";

export const store = configureStore({
  reducer: userReducer,
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

Просто передаем созданный нами `reducer` в `configureStore`. Далее создаем типы для работы с нашим `store`. Это делается для правильной типизации, чтобы мы могли использовать возможности `TypeScript`.

Также не забываем развернуть `Redux` в нашем приложении:

```tsx
// main.tsx
import { App } from "@app/routes";
import { ThemeProvider } from "@shared/index";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import React from "react";
import { Provider } from "react-redux";

import i18n from "i18next";
import { initReactI18next } from "react-i18next";
import LanguageDetector from "i18next-browser-languagedetector";

import { ruBaseDictionary, enBaseDictionary, kzBaseDictionary, Languages } from "@shared/index";
import { ruLoginDictionary, enLoginDictionary, kzLoginDictionary } from "@widgets/login";
import {
  ruLanguageSwitcherDictionary,
  enLanguageSwitcherDictionary,
  kzLanguageSwitcherDictionary,
} from "@widgets/languageSwitcher";
import { store, persistor } from "@app/store";
import { PersistGate } from "redux-persist/integration/react";

// Multingualism configuration
i18n
  .use(initReactI18next)
  .use(LanguageDetector)
  .init({
    resources: {
      [Languages.RUSSIAN]: {
        translation: { ...ruBaseDictionary, ...ruLoginDictionary, ...ruLanguageSwitcherDictionary },
      },
      [Languages.ENGLISH]: {
        translation: { ...enBaseDictionary, ...enLoginDictionary, ...enLanguageSwitcherDictionary },
      },
      [Languages.KAZAKHSTAN]: {
        translation: { ...kzBaseDictionary, ...kzLoginDictionary, ...kzLanguageSwitcherDictionary },
      },
    },
    fallbackLng: Languages.RUSSIAN,
    interpolation: {
      escapeValue: false,
    },
  });

const root = ReactDOM.createRoot(document.getElementById("root") as HTMLElement);
root.render(
  <Provider store={store}>
      <React.StrictMode>
        <ThemeProvider>
          <BrowserRouter>
            <App />
          </BrowserRouter>
        </ThemeProvider>
      </React.StrictMode>
  </Provider>,
);
```

3) Используем `Redux` в наших компонентах

```tsx
// UserProfileButton

import { RentangleBorderButton } from "@shared/ui/buttons";
import PersonIcon from "@mui/icons-material/Person";
import { useTheme } from "@shared/index";
import { LoginModal } from "@widgets/login";
import { useSelector } from "react-redux";
import type { RootState } from "@shared/store";
import { useNavigate } from "react-router";
import { useState } from "react";
import { PageLinks } from "@shared/pagesLinks";

export const UserProfileButton: React.FC = () => {
  const { theme } = useTheme();
  const [isSignUpFormOpen, setSignUpFormOpen] = useState<boolean>(false);
  
  // ========
  const user = useSelector((state: RootState) => {
    return state.user;
  });
  // ========
  
  const navigate = useNavigate();

  const handleButtonClick = () => {
    if (user.user) {
      navigate(PageLinks.PROFILE_PAGE);
    } else {
      setSignUpFormOpen(true);
    }
  };

  return (
    <>
      <RentangleBorderButton mainColor="dark" onClick={() => handleButtonClick()}>
        <PersonIcon
          sx={theme === "light" ? { color: "var(--color-base-light)" } : { color: "var(--color-base-dark)" }}
        />
      </RentangleBorderButton>
      <LoginModal open={isSignUpFormOpen} onClose={() => setSignUpFormOpen(false)} />
    </>
  );
};
```

Используем `useSelector()` чтобы получить наше текущее состояние пользователя. Как мы видим, мы передаем тип `RootState`, чтобы мы могли легко обратиться к компонентам, используя типизацию.

`dispatch` я использую в другом файле — `Login.tsx` (`feat` логина пользователя)

```tsx
import { type User } from "@entities/user";
import { clearUser, setUser } from "@entities/user/model";
import { UnauthorizedError, type Language } from "@shared/index";
import type { AppDispatch } from "@shared/store";
import { getTokens, type AccessTokenPayload, type RefreshTokenPayload } from "@shared/tokens";
import { decodeAccessToken, decodeRefreshToken } from "@shared/tokens";

export const LoginFeat = async (login: string, password: string, language: Language, dispatch: AppDispatch) => {
  try {
    const [accessToken, refreshToken] = await getTokens(login, password, language);

    const decodedAccessToken: AccessTokenPayload = decodeAccessToken(accessToken);
    const decodedRefreshToken: RefreshTokenPayload = decodeRefreshToken(refreshToken);

    const user: User = {
      user_id: decodedAccessToken.user_id,
      name: decodedRefreshToken.login,
      role: decodedAccessToken.role_id,
      avatarUrl: "TODO: set avatar in access token",
    };

	// =====
    dispatch(setUser(user));
    // ======
  } catch (error) {
    if (!(error instanceof UnauthorizedError)) {
      throw error;
    } else {
      // =====
      dispatch(clearUser());
      // =====
      console.error("Error when authorize", error);
      throw error;
    }
  }
};
```


На самом деле это не вся настройка для использования `Redux` для хранения и изменения состояния пользователя, ведь `Redux` хранит состояние в оперативной памяти, а значит, что перезапуске сессии (обновлении страницы) состояние потеряется. 
Чтобы состояния, такие как авторизирован ли пользователь, сохранять, нужно использовать расширение `redux-persist`. Описывается это в карточке: [[Redux persist]]