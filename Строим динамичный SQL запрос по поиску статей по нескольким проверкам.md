[[FastAPI.canvas|FastAPI]]
[[SQLAlchemy ORM]]

```python
    async def get_articles(
        self,
        # filters params
        language: LanguageEnum,
        category_id: tuple[ArticleCategoriesID, ...] | None = None,
        statuses: tuple[ArticleStatus, ...] | None = None,
        tags: tuple[int, ...] | None = None,
        # pagination params
        limit: int = DEFAULT_LIMIT,
        offset: int = 0,
        order_by: ArticleSortBy = ArticleSortBy.PUBLISHED_AT,
        order_direction: SortOrder = SortOrder.DESC,
    ) -> list[Article]:
        stmt = (
            select(
                A.c.article_id,
                AT.c.title,
                A.c.views_count,
                A.c.slug,
                A.c.status_id.label("status"),
                L.c.language_id.label("language"),
                BCT.c.blog_category_id,
                BCT.c.name.label("blog_category_name"),
                AT.c.image_src,
                A.c.published_at,
                func.nullif(
                    func.jsonb_agg(
                        func.jsonb_build_object(
                            "tag_id", TT.c.tag_id, "tag_name", TT.c.name
                        )
                    ),
                    cast("[null]", JSONB),
                ).label("tags"),
            )
            .select_from(
                A.join(AT, A.c.article_id == AT.c.article_id)
                .join(L, AT.c.language_id == L.c.language_id)
                .outerjoin(
                    BCT,
                    and_(
                        A.c.blog_category_id == BCT.c.blog_category_id,
                        L.c.language_id == BCT.c.language_id,
                    ),
                )
                .outerjoin(TA, A.c.article_id == TA.c.article_id)
                .outerjoin(
                    TT,
                    and_(
                        TT.c.language_id == L.c.language_id,
                        TT.c.tag_id == TA.c.tag_id,
                    ),
                )
            )
            .where(L.c.language_id == language)
            .group_by(
                A.c.article_id,
                AT.c.language_id,
                AT.c.article_id,
                L.c.language_id,
                BCT.c.language_id,
                BCT.c.blog_category_id,
            )
            .limit(limit)
            .offset(offset)
        )

        # ====== ====== ====== ====== ====== ====== ====== ====== ======
        # dynamic sql editing

        # 1. filtration
        if category_id:
            stmt = stmt.where(A.c.blog_category_id.in_(category_id))
        if statuses:
            stmt = stmt.where(A.c.status_id.in_(statuses))
        if tags:
            stmt = stmt.where(TT.c.tag_id.in_(tags))

        # 2. order by
        match order_by:
            case ArticleSortBy.VIEWS_COUNT:
                stmt = (
                    stmt.order_by(A.c.views_count.desc())
                    if order_direction == SortOrder.DESC
                    else stmt.order_by(A.c.views_count.asc())
                )
            case ArticleSortBy.PUBLISHED_AT:
                stmt = (
                    stmt.order_by(A.c.published_at.desc().nullsfirst())
                    if order_direction == SortOrder.DESC
                    else stmt.order_by(A.c.published_at.asc())
                )
        # ====== ====== ====== ====== ====== ====== ====== ====== ======

        try:
            async with self.__session as session:
                result = await session.execute(stmt)

            result = result.mappings().all()
            article_list = [
                Article(
                    article_id=article.article_id,
                    language=article.language,
                    title=article.title,
                    slug=article.slug,
                    status=article.status,
                    image_src=article.image_src,
                    views_count=article.views_count,
                    category=ArticleCategory(
                        category_id=article.blog_category_id,
                        name=article.blog_category_name,
                    )
                    if article.blog_category_id
                    else None,
                    published_at=article.published_at,
                    tags=[
                        Tag(
                            tag_id=tag["tag_id"],
                            name=tag["tag_name"],
                        )
                        for tag in article.tags
                        if tag and tag.get("tag_id")
                    ],
                )
                for article in result
            ]

            return article_list

        except IntegrityError as error:
            logger.error(
                "Integrity error of get article list with"
                + " (language, category, statuses_list, tags, limit, offset)"
                + " = (%s, %s, %s, %s, %s, %s)",
                language,
                category_id,
                statuses,
                tags,
                limit,
                offset,
                exc_info=error,
            )
            raise ArticleIntegrityError from error

        except DBAPIError as error:
            logger.error(
                "DBAPI error of get article list with"
                + " (language, category, statuses_list, tags, limit, offset)"
                + " = (%s, %s, %s, %s, %s, %s)",
                language,
                category_id,
                statuses,
                tags,
                limit,
                offset,
                exc_info=error,
            )
            raise ArticleDatabaseError from error


```