[[Frontend]]
[[Создаем запросы на ручки бэкэнда на фронтеде правильно. Routes to backend from frontend]]

Чтобы не блокировать нашу страничку, запросы на сторонние серверы лучше оборачивать в `waitingResponse`, который будет принимать хук и время, если за которое ответ не придет, выпадет ошибка. 

Дерево хуков:

```bash
├── hooks
│   ├── auth
│   │   └── useTelegramAuth.tsx
│   └── waitingResponse.tsx
```

`waitngResponse.tsx`

```tsx
const DEFAULT_TIMEOUT_EXCEPTION = 'Request timed out.';

// async function for waiting server response
export const withTimeout = async (
    promise: Promise<any>,
    ms: number,
    timeoutMessage: string = DEFAULT_TIMEOUT_EXCEPTION,
) => {
    const timeout = new Promise < any > (
        (_, reject) => {
            setTimeout(() => reject(new Error(timeoutMessage)), ms);
        });
    return Promise.race([promise, timeout])
};

```

`useTelegramAuth.tsx`

```tsx
import { useCallback, useState } from "react";
import { telegramLogin } from "../../api/auth/telegramLogin";
import { ERROR_MESSAGES } from "../../core/constants/errorsMessages";
import { DEFAULT_TIMEOUTS } from "../../core/constants/timeout_values";
import { NotTelegramWebAppError } from "../../errors/auth_errors";
import { ServerUnvailableErorr } from "../../errors/server_errors";
import { TelegramLoginResponse } from "../../schemas/api_schemas/authShemas";
import { TelegramWebApp } from "../../schemas/telegramSchemas";
import { withTimeout } from "../waitingResponse";

// Custom hook for handling Telegram authentication
export const useTelegramAuth = () => {
    // State to track authentication status: loading, success, or error
    const [authStatus, setAuthStatus] = useState<"loading" | "success" | "error">("loading");
    const [error, setError] = useState<string | null>(null);

    // Memoized function to send login request to backend
    const loginWithTelegram = useCallback(async (initData: string) => {
        try {
            // Call telegramLogin with timeout to prevent hanging
            // ========================
            // вот тут и вызываем наш таймер
            const response: TelegramLoginResponse = await withTimeout(
                telegramLogin(initData),
                DEFAULT_TIMEOUTS.API_REQUEST_TIMEOUT, // TODO: take timeouts from configuration file
                ERROR_MESSAGES.TIMEOUT_AUTH_ERROR
            );

            console.debug(response);

            // Check if login was successful
            if (response.status === "success") {
                // DO SOMETHING
            } else {
                throw new NotTelegramWebAppError(
                    response.message
                    || ERROR_MESSAGES.TELEGRAM_WEBAPP_ERROR
                );
            }
        } catch (error: any) {
            console.error(error);
            throw new ServerUnvailableErorr(
                error.response?.data?.detail
                || ERROR_MESSAGES.SERVER_ERROR
            );
        }
    }, []);

	// Остальной код ...
    return { pageStatus: authStatus, error, telegramAuthentificate };
};

```