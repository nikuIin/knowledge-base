[[SQLAlchemy]]
[[FastAPI.canvas|FastAPI]] — canvas

Есть проблема: если 1 таблица имеет 2 `foreign key` на другую таблицу, то в описании моделей `sqlalchemy` возникнет ошибка. Например, есть таблица `deal`, она ссылается 2 раза на таблицу `user`:
1. на менеджера
2. на лида

Вот это как решилось на уровне моделей `SQLAlchemy`:

```python

class User(Base, TimeStampMixin):
    __tablename__ = "user"

    user_id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid7,
        nullable=False,
    )
    login: Mapped[str] = mapped_column(
        String(255),
        nullable=False,
        unique=True,
    )
    email: Mapped[str | None] = mapped_column(
        VARCHAR(320),
        nullable=True,
    )
    password: Mapped[str] = mapped_column(
        String(255),
        nullable=False,
    )
    role_id: Mapped[int] = mapped_column(
        Integer,
        ForeignKey("role.role_id", ondelete="CASCADE"),
        nullable=False,
    )
    is_registered: Mapped[bool] = mapped_column(
        Boolean,
        default=False,
        nullable=False,
    )

    __table_args__ = (
        CheckConstraint("length(login) > 0", name="user_login_check"),
        CheckConstraint("length(password) > 0", name="user_password_check"),
        CheckConstraint("length(email) >= 3", name="user_email_check"),
    )

    role = relationship("Role", back_populates="users")
    refresh_tokens = relationship("RefreshToken", back_populates="user")
    md_user = relationship("MdUser", back_populates="user", uselist=False)
    author_translates = relationship("Author", back_populates="user")
    social_networks = relationship(
        "SocialNetwork",
        secondary="user_social_network",
        back_populates="users",
    )
    articles = relationship("Article", back_populates="author")

    lead_link = relationship(
	    "Deal",
	    back_populates="lead",
	    foreign_keys=lambda: [Deal.lead_id]
    )
    managed_deals = relationship(
	    "Deal",
	     back_populates="manager",
	    foreign_keys=lambda: [Deal.manager_id]
	)
	
    deal_histories = relationship("DealHistory", back_populates="manager")
    deal_messages = relationship("DealMessage", back_populates="user")




class Deal(Base, TimeStampMixin):
    __tablename__ = "deal"
    __table_args__ = (
        CheckConstraint(
            "deal_value::numeric >= 0", name="deal_value_positive"
        ),
        CheckConstraint(
            "probability between 0 and 1", name="deal_probability_range"
        ),
        CheckConstraint(
            "priority between -1 and 10", name="deal_priority_range"
        ),
    )

    deal_id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True),
        primary_key=True,
    )
    sale_stage_id: Mapped[int] = mapped_column(
        Integer,
        ForeignKey("sale_stage.sale_stage_id", ondelete="CASCADE"),
        nullable=False,
    )
    lead_id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True),
        ForeignKey("user.user_id", ondelete="CASCADE"),
        nullable=False,
    )
    cost: Mapped[Numeric] = mapped_column(
        NUMERIC,
        nullable=False,
    )
    probability: Mapped[Numeric] = mapped_column(
        NUMERIC(3, 2),
        nullable=False,
        default=0,
    )
    fields: Mapped[dict | None] = mapped_column(
        JSONB,
        nullable=True,
    )
    priority: Mapped[int | None] = mapped_column(
        Integer,
        nullable=True,
    )
    lost_reason_id: Mapped[int | None] = mapped_column(
        Integer,
        ForeignKey("lost_reason.lost_reason_id", ondelete="SET NULL"),
        nullable=True,
    )
    lost_reason: Mapped[str | None] = mapped_column(
        Text,
        nullable=True,
    )
    manager_id: Mapped[uuid.UUID | None] = mapped_column(
        UUID(as_uuid=True),
        ForeignKey("user.user_id", ondelete="SET NULL"),
        nullable=True,
    )
    close_at: Mapped[datetime | None] = mapped_column(
        TIMESTAMP(timezone=True),
        nullable=True,
    )

    sale_stage = relationship("SaleStage", back_populates="deals")
    lost_reason = relationship("LostReason", back_populates="deals")
    manager = relationship(
	    "User",
	    back_populates="managed_deals",
	    foreign_keys=[manager_id]
	)
    deal_histories = relationship("DealHistory", back_populates="deal")
    lead= relationship(
	    "User",
	    back_populates="lead_link",
	    foreign_keys=[lead_id]
	)
    deal_messages = relationship("DealMessage", back_populates="deal")

```
