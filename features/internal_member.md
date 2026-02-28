# Internal Structure Members

 - Feature ID: F004

This feature imposes a soft restriction on access to structure members. Any structure members prefixed with an underscore (`_`) are regarded as _internal_ and are advised to be accessed only by [function members](./func_in_struct.md), [function alias members](./switch_in_struct.md), or annotated functions. Failing to satisfy this results in a compiler warning.
