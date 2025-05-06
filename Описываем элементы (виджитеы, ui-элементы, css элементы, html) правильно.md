[[CSS]]
[[Frontend]]
[[tailwind]]

Шаги правильного описания `ui-элемента`:

1. Определить `props` (все что может быть у элемента):
```tsx
type StatsCardProps = {
	// а это все, что может быть внутри элемента, является
	// например в <StatsCard>image.png</StatsCardProps>, image.png — children
	children: React.ReactNode,
    wins: number;
    losses: number;
    onClick?: () => void;
    to?: string,
    variant?: Variant; // разные стили (ну, например кнопки на сайте бывают разных цветов)
}

type Variant = "primary" | "secondary"; // описываем название вариантов

const variantStyles: Record<Variant, string> = {
	primary: "bg-green-500 hover:...", // используем tailwind
	secondary: "bg-black-100 hover:...", 
};

export const StatsCard: React.FC<StatsCardProps> = ({
	children,
	wins = 0,
	losses = 0,
	to,
	onClick,
	variant = "primary"
}) => {
	const navigate = useNavigate()

	const handleClick() = () => {
		if(to){
			navigate(to)
		}
		else if(onClick){
			onClick();
		}
	};

	return (
		<div 
		   onClick={handleClick}
		   className={`${variantStyles[variant]} text-[18px] font-semibold`}
			{children}
		</div>
	);
};


```