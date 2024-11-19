[[DynamoDB]] is fast and incredibly scalable, but lacks a lot of common features you would expect from a more popular database like [[PostgreSQL]]. One such issue is the absence of a good Object Document Mapper

I use [[FastAPI]], which has great support for [[Pydantic]], so it would be *really* nice to define my table schemas using Pydantic models like [SQLModel](https://sqlmodel.tiangolo.com/) does.

- Schema description with type-checking support, like [[PynamoDB]]
- Asyncio support, like [asyncpg](https://github.com/MagicStack/asyncpg) has

Possible solution shapes:
- Build a [custom SQLAlchemy Dialect](https://docs.sqlalchemy.org/en/13/core/internals.html#sqlalchemy.engine.default.DefaultDialect) to support DynamoDB
- Monkeypatch [[PynamoDB]] with asyncio support
- Build from scratch using Pydantic and aioboto3