[[Frontend]]
[[CSS]]
[[Описываем элементы (виджитеы, ui-элементы, css элементы, html) правильно]]

```tsx
import { useNavigate } from 'react-router';

// variants of using button styles. (participate — to take part of the tournament)
type Variant = "primary" | "off" | "canceled" | "participate";

// Описываем (schema) кнопки
type ButtonProps = {
    children: React.ReactNode;
    onClick?: () => void;
    variant?: Variant;
    to?: string;
};


const variantStyles: Record<Variant, string> = {
    primary: "bg-green-500 hover:bg-green-600 cursor-pointer text-white",
    off: "bg-gray-500 over:bg-gray-600 cursor-pointer text-white",
    canceled: "bg-red-500 over:bg-red-600 cursor-pointer text-white",
    participate: "bg-blue-500 hover:bg-blue-600 cursor-pointer text-white",
};

export const Button: React.FC<ButtonProps> = ({ children, onClick, to, variant = "primary" }) => {

    const navigate = useNavigate()
    // Combine navigation and custom click handler (to > onClick)
    const handleClick = () => {
        if (to) {
            navigate(to); // Navigate to the specified path
        }
        if (onClick) {
            onClick(); // Execute custom click handler if provided
        }
    };

    return (
        <button
            onClick={handleClick}
            className={`${variantStyles[variant]} text-[18px] font-semibold py-3.5 px-6 rounded-2xl w-full shadow`}
        >
            {children}
        </button>
    );
};

```