# Breaking Changes

This is a comprehensive list of the breaking changes introduced in the minor version releases while the action is on `0.x`.

## Versions

- [Version 0.3.x](#version-03x)

## Version 0.3.x

### One Channel And One Platform

`channel` takes one channel instead of a list, and `platform` takes one platform instead of `ios,android`. One run makes one release. On React Native and Expo run the action once per platform, for example through a matrix:

```diff
- with:
-   channel: staging,production
+ strategy:
+   matrix:
+     platform: [ios, android]
+ with:
+   channel: production
+   platform: ${{ matrix.platform }}
```

### Renamed `releases` Output

The `releases` output has been renamed to `release` and holds one object, the created release, or on a dry run the audience. `release-id` and `bundle-number` read that object:

```diff
- ${{ fromJson(steps.release.outputs.releases)[0].id }}
+ ${{ steps.release.outputs.release-id }}
```

### CLI Version

`cli-version` defaults to **0.3.0**, the lowest version the API accepts. A pinned older version is refused with `client_update_required`.
