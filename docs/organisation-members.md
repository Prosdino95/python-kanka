# Organisation Members

Characters are linked to organisations through a **membership**, a small record that carries the character, the organisation, and an optional role. Memberships are a sub-resource of organisations, available only on `client.organisations`.

## Important: this is not a field

An organisation payload contains a `members` key, but it is read-only. Sending it in a create or update call is accepted by the API and silently ignored — no error, no change:

```python
# Does nothing. The call succeeds and the member list stays empty.
client.organisations.update(2, name="I Veglianti", members=[10])
```

Use the methods below instead.

## Important: id, not entity_id

Unlike [posts](posts.md), which use the universal `entity_id`, memberships use **type-specific IDs** on both sides:

```python
organisation = client.organisations.get(2)
character = client.characters.get(10)

print(character.id)         # 10  — use this one
print(character.entity_id)  # 58  — NOT this one

client.organisations.add_member(organisation, character.id, role="Sovrintendente")
```

Passing an `entity_id` for the character raises a `ValidationError` with `The selected character id is invalid`. Passing one that happens to exist as a type-specific ID links the wrong character, so resolve IDs explicitly rather than reusing a number you have lying around.

## Listing members

```python
members = client.organisations.list_members(organisation)

for member in members:
    print(f"character {member.character_id}: {member.role}")
```

Supports pagination through `page` and `limit`.

## Adding a member

```python
member = client.organisations.add_member(
    organisation,           # or its type-specific ID
    character.id,
    role="Sovrintendente",  # optional
    is_private=False,       # optional
)
print(member.id)  # the membership ID, not the character's
```

## Updating a member

Only the fields you pass are sent, so a role can be changed on its own:

```python
client.organisations.update_member(organisation, member.id, role="Guardiano")
```

## Removing a member

This deletes the membership only. The character itself is untouched.

```python
client.organisations.remove_member(organisation, member.id)
```

## The OrganisationMember model

| Field | Description |
| --- | --- |
| `id` | Membership ID — the handle for update and remove |
| `organisation_id` | Organisation's type-specific ID |
| `character_id` | Character's type-specific ID |
| `role` | Role in the organisation, or `None` |
| `is_private` | Whether the membership is private |
| `status_id`, `pin_id`, `parent_id` | Optional membership metadata |
| `created_at`, `created_by`, `updated_at`, `updated_by` | Audit fields |
