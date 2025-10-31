# Sysrepo Compatibility Notes

## Sysrepo 3.x/4.x (SO Version 7-8) Compatibility

This Python wrapper has been updated to work with the upcoming sysrepo 3.x/4.x series
(currently in the devel branch). The changes from sysrepo 2.x to 3.x/4.x include
several function renames and API improvements.

### API Changes Implemented

Based on the compatibility notes in sysrepo's `compatibility/6.0.0_to_7.0.0/CHANGES`,
the following function renames have been incorporated:

#### Renamed Functions (Already Updated)

- `sr_event_notif_send()` → `sr_notif_send()` ✓
- `sr_event_notif_send_tree()` → `sr_notif_send_tree()` ✓
- `sr_event_notif_subscribe()` → `sr_notif_subscribe()` ✓
- `sr_event_notif_subscribe_tree()` → `sr_notif_subscribe_tree()` ✓
- `sr_oper_get_items_subscribe()` → `sr_oper_get_subscribe()` ✓
- `sr_process_events()` → `sr_subscription_process_events()` ✓
- `sr_get_context()` → `sr_acquire_context()` and `sr_release_context()` ✓
- `sr_get_module_access()` → `sr_get_module_ds_access()` ✓

#### Removed Functions (Not Used)

The following functions were removed in sysrepo 3.x but were not used by this wrapper:

- `sr_cancel_update_module()`
- `sr_connection_count()`
- `sr_get_module_info()` (now replaced with different mechanism)

#### Removed Flags (Not Used)

- `SR_SUBSCR_CTX_REUSE` - This flag was removed. The new API requires that subscription
  pointers be initialized to NULL on first call. This wrapper already does this correctly
  using `ffi.new("sr_subscription_ctx_t **")`.

### New Functions Not Yet Exposed

The following new functions are available in sysrepo 3.x/4.x but not yet exposed by this wrapper:

- `sr_acquire_data()` / `sr_session_acquire_data()` - Helper functions for context-safe data access
- `sr_install_module2()` - Extended module installation with more options
- `sr_install_modules2()` - Batch module installation with more options
- `sr_get_module_replay_support()` - Query notification replay support
- `sr_check_module_ds_access()` - Check datastore access permissions
- `sr_get_su_uid()` - Get sysrepo superuser UID
- `sr_subscription_thread_suspend()` / `sr_subscription_thread_resume()` - Thread control

These functions can be added in future versions if needed.

### Changed Function Signatures

#### `sr_get_module_ds_access()`

Changed from:
```c
int sr_get_module_ds_access(sr_conn_ctx_t *conn, const char *module_name, 
                            sr_datastore_t datastore, char **owner, char **group, mode_t *perm);
```

To:
```c
int sr_get_module_ds_access(sr_conn_ctx_t *conn, const char *module_name, 
                            int mod_ds, char **owner, char **group, mode_t *perm);
```

The `sr_datastore_t` enum was replaced with a plain `int` to allow for more flexible
datastore specification. This change is already reflected in `cffi/cdefs.h`.

### Context Management Changes

The biggest change is how the libyang context is accessed:

- **Old API**: `sr_get_context()` returned a const context pointer directly
- **New API**: `sr_acquire_context()` acquires a READ LOCK on the context and must be
  paired with `sr_release_context()` to release the lock

This is important because the context can be changed at any point (e.g., when modules
are installed/removed), so locking ensures safe concurrent access.

The wrapper properly implements this pattern using context managers:
```python
with conn.get_ly_ctx() as ctx:
    # use ctx safely
# context is automatically released
```

### Testing Compatibility

To test with the sysrepo devel branch:

```bash
SYSREPO_BRANCH=devel LIBYANG_BRANCH=devel python3 -m tox -e py312
```

The `tox-install.sh` script will clone and build the specified branches of libyang
and sysrepo before running tests.

### Migration from Older Versions

If you're using an older version of sysrepo-python with sysrepo 2.x, no code changes
should be required. The wrapper maintains backward compatibility with sysrepo 2.2.0+.

### Version Support Matrix

| sysrepo-python | Sysrepo C Library | SO Version | Notes |
|----------------|-------------------|------------|-------|
| 2.x.x (current) | 2.2.0 - 4.x.x | 6 - 8 | Compatible with both stable and devel |
| 1.x.x | 1.x.x | < 6 | EOL, use sysrepo-python 0.7.0 |
| 0.7.0 | 1.x.x | < 6 | Last version for sysrepo 1.x |

### Additional Resources

- [Sysrepo Repository](https://github.com/sysrepo/sysrepo)
- [Sysrepo Devel Branch](https://github.com/sysrepo/sysrepo/tree/devel)
- [Sysrepo Compatibility Documentation](https://github.com/sysrepo/sysrepo/tree/devel/compatibility)
