# fivetran-catfritz/release_tester v0.2.0
[PR #43](https://github.com/fivetran/dbt_zendesk_source/pull/43) introduces the following updates:

## Feature Updates
- Added the `internal_user_criteria` variable, which can be used to mark internal users whose `USER.role` may have changed from `agent` to `end-user` after they left your organization. This variable accepts SQL that may reference any non-custom field in `USER`, and it will be wrapped in a `case when` statement in the `stg_zendesk__user` model.
  - Example usage:
```yml
# dbt_project.yml
vars:
  zendesk_source:
    internal_user_criteria: "lower(email) like '%@fivetran.com' or external_id = '12345' or name in ('Garrett', 'Alfredo')" # can reference any non-custom field in USER
```
  - Output: In `stg_zendesk__user`, users who match your criteria and have a role of `end-user` will have their role switched to `agent`. This will ensure that downstream SLA metrics are appropriately calculated.

## Under the Hood
- Updated the way we dynamically disable sources. Previously, we used a custom `meta.is_enabled` flag, but, since we added this, dbt-core introduced a native `config.enabled` attribute.  We have opted to use the dbt-native config instead.
- Updated the pull request [templates](/.github).
- Included auto-releaser GitHub Actions workflow to automate future releases.

# fivetran-catfritz/release_tester v0.1.1
## Features
- Addition of the `pinterest__using_keywords` (default=`true`) variable that allows users to disable the relevant keyword reports in the downstream Pinterest models if they are not used. ([PR #23](https://github.com/fivetran/dbt_pinterest_source/pull/23))

## Under the Hood:
- Incorporated the new `fivetran_utils.drop_schemas_automation` macro into the end of each Buildkite integration test job. ([PR #22](https://github.com/fivetran/dbt_pinterest_source/pull/22))
- Updated the pull request [templates](/.github). ([PR #22](https://github.com/fivetran/dbt_pinterest_source/pull/22))

# dbt_pinterest_source v0.7.11
## 🔧 Fixes
- Added `pin_promotion_id` to unique-combination-of-columns test for model `stg_pinterest_ads` to remedy test failures. ([#21](https://github.com/fivetran/dbt_pinterest_source/pull/21))
# dbt_pinterest_source v0.7.0

## 🚨 Breaking Changes 🚨:
[PR #19](https://github.com/fivetran/dbt_pinterest_source/pull/19) includes the following breaking changes:
- Dispatch update for dbt-utils to dbt-core cross-db macros migration. Specifically `{{ dbt_utils.<macro> }}` have been updated to `{{ dbt.<macro> }}` for the below macros:
    - `any_value`
    - `bool_or`
    - `cast_bool_to_text`
    - `concat`
    - `date_trunc`
    - `dateadd`
    - `datediff`
    - `escape_single_quotes`
    - `except`
    - `hash`
    - `intersect`
    - `last_day`
    - `length`
    - `listagg`
    - `position`
    - `replace`
    - `right`
    - `safe_cast`
    - `split_part`
    - `string_literal`
    - `type_bigint`
    - `type_float`
    - `type_int`
    - `type_numeric`
    - `type_string`
    - `type_timestamp`
    - `array_append`
    - `array_concat`
    - `array_construct`
- For `current_timestamp` and `current_timestamp_in_utc` macros, the dispatch AND the macro names have been updated to the below, respectively:
    - `dbt.current_timestamp_backcompat`
    - `dbt.current_timestamp_in_utc_backcompat`
- Dependencies on `fivetran/fivetran_utils` have been upgraded, previously `[">=0.3.0", "<0.4.0"]` now `[">=0.4.0", "<0.5.0"]`.

# dbt_pinterest_source v0.6.0

## 🚨 Breaking Changes 🚨
- The `pin_promotion_report_pass_through_metric` variable has been renamed to `pinterest__pin_promotion_report_passthrough_metrics`. ([#18](https://github.com/fivetran/dbt_pinterest_source/pull/18))
- The declaration of passthrough variables within your root `dbt_project.yml` has changed. To allow for more flexibility and better tracking of passthrough columns, you will now want to define passthrough metrics in the following format: ([#18](https://github.com/fivetran/dbt_pinterest_source/pull/18))
> This applies to all passthrough metrics within the `dbt_pinterest_source` package and not just the `pinterest__pin_promotion_report_passthrough_metrics` example.
```yml
vars:
  pinterest__pin_promotion_report_passthrough_metrics:
    - name: "my_field_to_include" # Required: Name of the field within the source.
      alias: "field_alias" # Optional: If you wish to alias the field within the staging model.
```
## 🎉 Feature Enhancements 🎉
PR [#18](https://github.com/fivetran/dbt_pinterest_source/pull/18) includes the following changes:
- Addition of the following staging models which pull from the source counterparts. The inclusion of the additional `_report` source tables is to generate a more accurate representation of the Pinterest Ads data. For example, not all Ad types are included within the `pin_promotion_report` table. Therefore, the addition of the further grain reports will allow for more flexibility and accurate Pinterest Ad reporting. 
  - `stg_pinterest_ads__ad_group_report`
  - `stg_pinterest_ads__advertiser_report`
  - `stg_pinterest_ads__campaign_report`
  - `stg_pinterest_ads__keyword_report`
  - `stg_pinterest_ads__advertiser_history`
  - `stg_pinterest_ads__keyword_history`

- Inclusion of additional passthrough metrics: 
  - `pinterest__ad_group_report_passthrough_metrics`
  - `pinterest__campaign_report_passthrough_metrics`
  - `pinterest__advertiser_report_passthrough_metrics`
  - `pinterest__keyword_report_passthrough_metrics`

- README updates for easier navigation and use of the package. 
- Addition of identifier variables for each of the source tables to allow for further flexibility in source table direction within the dbt project.
- Included grain uniqueness tests for each staging table. 

## Contributors
- [@bnealdefero](https://github.com/bnealdefero) ([#18](https://github.com/fivetran/dbt_pinterest_source/pull/18))

# dbt_pinterest_source v0.5.0
🎉 dbt v1.0.0 Compatibility 🎉
## 🚨 Breaking Changes 🚨
- Adjusts the `require-dbt-version` to now be within the range [">=1.0.0", "<2.0.0"]. Additionally, the package has been updated for dbt v1.0.0 compatibility. If you are using a dbt version <1.0.0, you will need to upgrade in order to leverage the latest version of the package.
  - For help upgrading your package, I recommend reviewing this GitHub repo's Release Notes on what changes have been implemented since your last upgrade.
  - For help upgrading your dbt project to dbt v1.0.0, I recommend reviewing dbt-labs [upgrading to 1.0.0 docs](https://docs.getdbt.com/docs/guides/migration-guide/upgrading-to-1-0-0) for more details on what changes must be made.
- Upgrades the package dependency to refer to the latest `dbt_fivetran_utils`. The latest `dbt_fivetran_utils` package also has a dependency on `dbt_utils` [">=0.8.0", "<0.9.0"].
  - Please note, if you are installing a version of `dbt_utils` in your `packages.yml` that is not in the range above then you will encounter a package dependency error.

# dbt_pinterest_source v0.1.0 -> v0.4.0
Refer to the relevant release notes on the Github repository for specific details for the previous releases. Thank you!
