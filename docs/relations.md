# Relations

Relations are directed links between two entities: "Capomastro di", "in conflitto con", "allea con". They are a sub-resource of the entity that owns them, available on every entity manager.

## Important: entity_id on both sides

Relations use the universal `entity_id` for both the owner and the target — unlike [organisation members](organisation-members.md), which use type-specific IDs:

```python
character = client.characters.get(1)
location = client.locations.get(2)

client.characters.create_relation(
    character,              # or character.entity_id
    location.entity_id,     # the target, as an entity_id
    "Capomastro di",
)
```

Since the endpoint is entity-generic, any manager can drive it: `client.characters.list_relations(entity_id)` works for a location too. Use the manager of the owning entity for readability.

## Listing relations

Only the relations an entity **owns** are returned. Relations pointing at it from elsewhere belong to the other entity.

```python
for relation in client.characters.list_relations(character):
    print(f"{relation.relation} -> entity {relation.target_id}")
```

## Creating a relation

```python
relation = client.characters.create_relation(
    character,
    target_id=location.entity_id,
    relation="Capomastro di",
    attitude=50,          # optional, may be negative
    visibility_id=2,      # optional, 2 = admin only
)
```

### One direction, or two

Relations are one-directional by default. Pass `two_way=True` to also create the mirror on the target:

```python
relation = client.characters.create_relation(
    character, other.entity_id, "allea con", two_way=True
)
print(relation.mirror_id)  # the relation created on the other side
```

The field that does this is `two_way`. **`is_mirrored` is accepted by the API and does nothing** — no error, no mirror.

Note that the API answers a create with *every* relation the entity owns, not just the new one. The library returns the newest match on target and label.

## Updating a relation

Only the fields you pass are sent, and the mirror is not touched:

```python
client.characters.update_relation(character, relation.id, relation="in conflitto con")
```

## Deleting a relation

```python
client.characters.delete_relation(character, relation.id)
```

This deletes **one side only**. If the relation had a mirror, the mirror survives with its `mirror_id` cleared, leaving half a link in place. To remove both:

```python
relation = next(r for r in client.characters.list_relations(character) if r.id == rid)
client.characters.delete_relation(character, relation.id)
if relation.mirror_id:
    client.characters.delete_relation(relation.target_id, relation.mirror_id)
```

## The Relation model

| Field | Description |
| --- | --- |
| `id` | Relation ID — the handle for update and delete |
| `owner_id` | Entity the relation belongs to, as an entity_id |
| `target_id` | Entity it points at, as an entity_id |
| `relation` | The label |
| `attitude` | Numeric attitude towards the target |
| `visibility_id` | 1 = all, 2 = admin only |
| `mirror_id` | The relation on the other side, when `two_way` was used |
| `colour`, `is_star`, `is_pinned` | Display options |
| `created_at`, `created_by`, `updated_at`, `updated_by` | Audit fields |
