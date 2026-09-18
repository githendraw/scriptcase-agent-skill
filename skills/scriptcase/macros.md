# ScriptCase Blank App — Built-in Macro & Runtime Helper Catalog

Reference list of ScriptCase macros usable from a **Blank application** (`onExecute`).
Only general-purpose macros are listed; grid/form/button-specific macros do not apply to blank apps.

> **Caveats (read first)**
> - A Blank app runs only the `onExecute` event, so only general macros take effect.
> - The connection argument, when present, must be a **literal string** (never a variable or `[global]`).
> - Macro signatures can differ between ScriptCase versions. The code editor shows the exact
>   signature via **Ctrl+Space** — verify there before relying on a signature.
> - Not every macro is needed; use only what a given app requires.

---

## A. Data & SQL

| Macro | Purpose |
|-------|---------|
| `sc_lookup(Dataset, "SQL", "Connection")` | SELECT → array dataset (`{ds[0][0]}`) |
| `sc_select(dataset, "SQL", "Connection")` | SELECT → recordset (`->fields`, `->EOF`) |
| `sc_exec_sql("SQL", "Connection")` | Execute INSERT/UPDATE/DELETE/DDL |
| `sc_lookup_field(Dataset, "SQL", "Connection")` | Variant of `sc_lookup`; confirm return shape in the editor |
| `sc_set_fetchmode(parm)` | Change the dataset fetch/return mode |
| `sc_concat()` | Concatenate strings/fields in SELECT across databases |
| `sc_begin_trans("Connection")` | Start a transaction |
| `sc_commit_trans("Connection")` | Commit a transaction |
| `sc_rollback_trans("Connection")` | Roll back a transaction |
| `sc_change_connection("Old", "New")` | Switch connection at runtime |
| `sc_connection_edit("Name", $arr)` | Edit an existing connection at runtime |
| `sc_connection_new("Name", $arr)` | Create a new connection dynamically |
| `sc_reset_change_connection` | Undo `sc_change_connection` |
| `sc_reset_connection_edit` | Undo `sc_connection_edit` |
| `sc_reset_connection_new` | Undo `sc_connection_new` |
| `sc_sql_protect(Value, "Type", "Connection")` | DB-aware escape/protect a value |
| `sc_sql_injection({Field})` or `($var)` | Protect a field/variable against SQL injection |
| `sc_where_current` / `sc_where_orig` / `sc_where_filter` | WHERE clause helpers (mostly grid-oriented) |

## B. Error, Log & Warning

| Macro | Purpose |
|-------|---------|
| `sc_error_message("Text")` | Register an application error message |
| `sc_error_exit("URL/App", "Target")` | Stop execution if `sc_error_message` was called |
| `sc_error_continue("Event")` | Disable ScriptCase's default DB error handling for an event |
| `sc_warning 'on'` / `sc_warning 'off'` | Enable/disable warning message control |
| `sc_log_add("action", "description")` | Insert a row into the log table |
| `sc_log_split({description})` | Parse the log description into an array |

## C. Session / Global & Application Defaults

| Macro | Purpose |
|-------|---------|
| `sc_set_global($variable)` or `({Field})` | Register a session/global variable |
| `sc_reset_global([g1], [g2])` | Remove session variables |
| `sc_apl_default("app", "type")` / `sc_reset_apl_default` | Set/reset the initial application when the session is lost |
| `sc_apl_conf("app", "property", "value")` / `sc_reset_apl_conf("app", "property")` | Change another application's execution property |
| `sc_apl_status("app", "status")` / `sc_reset_apl_status` | Enable/disable an application at user level |

## D. Navigation & Messages (JavaScript)

| Macro | Purpose |
|-------|---------|
| `sc_alert("Message", $array)` | Show a JS alert/toast |
| `sc_confirm("Message")` | Show a JS confirm dialog |
| `sc_ajax_message("Message", "Title", "Parameters", "Parameters_Redir", "String_toast")` | Custom AJAX message |
| `sc_ajax_javascript('Method', array("param"))` | Run a JS method |
| `sc_ajax_refresh` | Refresh via AJAX |
| `sc_redir('app/url', params, 'target', 'error', 'modal_height', 'modal_width')` | Redirect to an application/URL |
| `sc_url_exit("URL")` / `sc_exit(Option)` | Change exit URL / force exit |
| `sc_make_link("Application", Parameters)` / `sc_link(...)` | Build a link to another application |
| `sc_send_notification('title', 'message', 'destiny_type', 'to', 'from', 'link', 'dtexpire', 'profile')` | Send a notification |

## E. Include & Library

| Macro | Purpose |
|-------|---------|
| `sc_include("File", "Source")` | Include PHP routines |
| `sc_include_lib("Lib1", "Lib2", ...)` | Select application libraries dynamically |
| `sc_include_library("Target", "Library", "File", "include_once", "Require")` | Include a PHP file from a library |
| `sc_url_library("Target", "Library", "File")` | Return the URL/path of a library file |

## F. Date, Number, Text, Encoding & Language

| Macro | Purpose |
|-------|---------|
| `sc_date(Date, "Format", "Operator", D, M, Y)` | Add/subtract on dates |
| `sc_date_conv({Field}, "Input_Format", "Output_Format")` | Convert a date between formats |
| `sc_date_dif({D1}, "F1", {D2}, "F2")` | Difference between dates (days) |
| `sc_date_dif_2({D1}, "F1", {D2}, "F2", Option)` | Difference as days/months/years |
| `sc_date_empty({Field})` | True if a date field is empty |
| `sc_time_diff({dt1}, "F1", {dt2}, "F2")` | Difference in hours/minutes/seconds |
| `sc_format_num({Field}, ...)` / `sc_format_num_region({Field}, ...)` | Format numbers |
| `sc_trunc_num({Field}, Decimal_Number)` | Truncate to N decimals |
| `sc_calc_dv(Digit, Rest, Value, Module, Weights, Type)` | Check-digit (checksum) calculation |
| `sc_encode({Field})` / `sc_decode({Field})` | Encrypt / decrypt a value |
| `sc_get_language` / `sc_get_regional` / `sc_get_theme` / `sc_language` | Read current language/regional/theme |
| `sc_set_language('x')` / `sc_set_regional('x')` / `sc_set_theme('x')` | Change language/regional/theme |

## G. Email / SMS / Web Service / API

| Macro | Purpose |
|-------|---------|
| `sc_mail_send(SMTP, Usr, Pw, From, To, Subject, Message, Mens_Type, Copies, Copies_Type, Port, Connection_Type, Attachment, SSL)` | Send an email |
| `sc_send_mail_api($arr_settings)` | Send mail via API (e.g. Mandrill / Amazon SES) |
| `sc_send_sms($arr_settings)` | Send SMS via API |
| `sc_webservice("Method", "URL", "Port", "Send Method", $params, $settings, Timeout, Return)` | Call a web service |
| `sc_call_api($profile, $arr_settings)` | Call a configured API profile |
| `sc_api_upload(...)` / `sc_api_download(...)` / `sc_api_storage_delete(...)` | Cloud storage file operations |
| `sc_api_gc_get_url($app, $json_oauth)` / `sc_api_gc_get_obj($app, $json_oauth, $auth_code)` | Google Cloud OAuth helpers |

## H. File / ZIP

| Macro | Purpose |
|-------|---------|
| `sc_zip_file("File", "Zip")` | Create a ZIP from files/directories |
| `sc_seq_register` | Get a sequential register number |

## I. Authentication / LDAP / Security

| Macro | Purpose |
|-------|---------|
| `sc_ldap_login($server, $version, $user, $password, $dn, $group, $port, $library)` | Log in to LDAP |
| `sc_ldap_logout()` | Release the LDAP connection |
| `sc_ldap_search($filter = 'all', $attributes = array())` | Search LDAP |
| `sc_ldap_users($filter = 'all', $attributes = array())` | List LDAP users |
| `sc_ldap_groups` | List LDAP groups |
| `sc_user_logout('variable_name', 'variable_content', 'apl_redir.php', 'target')` | Log the user out |
| `sc_site_ssl` | Whether the site is under HTTPS |

---

## Runtime objects & special variables (not macros)

| Item | Use |
|------|-----|
| `$this->Db` | ADODB-style connection: `->Execute($sql)`, `->ErrorMsg()`, `->Close()` |
| `$this->Ini` | Application settings: `path_prod`, `path_imagens`, `Nm_lang`, `nm_tpbanco`, … |
| `$_POST`, `$_GET`, `$_SESSION`, `$_SERVER`, `$_REQUEST` | Standard PHP superglobals |
| `[global]` | ScriptCase global/session variable |
| `{composite}` | ScriptCase composite/field variable (scalar or array) |
| `$local` | Ordinary PHP local variable |

### Minimal safe patterns
```php
// Raw recordset (bypasses the sc_ macros) when you need full control
$rs = $this->Db->Execute("SELECT * FROM items WHERE id = " . (int)$id);
if ($rs !== false) {
    while (!$rs->EOF) {
        $name = $rs->fields['name'];
        $rs->MoveNext();
    }
    $rs->Close();
}

// Redirect to another application
sc_redir("other_app", "id=" . urlencode($id));

// Message + stop
sc_error_message("Invalid data.");
sc_error_exit();
```

---

## Not applicable to Blank apps (grid / form / button only)

`sc_field_display`, `sc_field_readonly`, `sc_field_disabled`, `sc_field_style`, `sc_field_color`,
`sc_field_init_off`, `sc_field_no_validate`, `sc_label`, `sc_btn_*`, `sc_block_display`,
`sc_form_show`, `sc_captcha_display`, `sc_event_hint`, `sc_set_focus`, `sc_set_export_name`,
`sc_set_pdf_name`, `sc_select_field`, `sc_select_order`, `sc_select_where`, `sc_groupby_*`,
`sc_get_groupby_rule`, `sc_hide_groupby_rule`, `sc_groupby_label`, `sc_master_value`,
`sc_changed`, `sc_get_wizard_step`, `sc_head_hide`, `sc_foot_hide`, `sc_widget_*`,
`sc_actionbar_*`, `sc_change_css`, `sc_text_style`, `sc_statistic`.

Sources: ScriptCase manual — *Macros* and *Macro × Applications × Events*.
