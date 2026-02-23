# Internal Structure Members

This feature imposes a soft restriction on access to structure members. Any structure members prefixed with an underscore (`_`) are regarded as _internal_ and are advised to be accessed only by non-internal [function members](./func_in_struct.md), non-internal [function alias members](./switch_in_struct.md), or annotated functions. Failing to satisfy this results in a compiler warning.
