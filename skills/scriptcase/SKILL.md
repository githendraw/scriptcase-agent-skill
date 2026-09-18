---
name: scriptcase
license: MIT
description: "Use when the user develops a ScriptCase BLANK application (onExecute) from a local .php draft that will be pasted into ScriptCase, or edits/debugs such code. Covers the no-<?php draft format, global variables [var] (session/Input-Output), composite/field variables {var}, local $var, and the SQL macros sc_lookup(), sc_select(), sc_exec_sql(), plus form POST insert/update, transactions, error/alert/redirect macros, connections, security, and common pitfalls. Trigger on: ScriptCase, blank app, onExecute, sc_lookup, sc_select, sc_exec_sql, [variable], {rs[0][0]}, {rs}->EOF, paste ke ScriptCase."
---

# ScriptCase Blank Application — User Development Guide

This skill is about **how the user writes the code** for a ScriptCase **Blank application**
(the only event is `onExecute`). The workflow is:

1. Write a plain PHP draft locally (e.g. `D:\KODING\...\myapp.php`), following the conventions below.
2. Copy/paste the whole draft into ScriptCase → Blank application → `onExecute` event.
3. ScriptCase generates **1 file → 1 folder** (e.g. `sc_pub_.../myapp/index.php`). Do not edit the
   generated output; always edit the draft/source and re-publish.

The macros (`sc_lookup`, `sc_select`, `sc_exec_sql`, …) **do not exist in plain PHP**. The draft
cannot be run with a normal PHP server; it only runs inside ScriptCase. Locally you can only sanity
check braces/quotes (a plain `php -l` will choke on `[var]` / `{var}` — that is expected).

A complete, generic copy-ready draft lives in `reference.md` in this skill folder.

---

## 1. Draft file format (CRITICAL)

`onExecute` already provides the opening `<?php`. Therefore the draft:

- **starts with plain PHP statements** — NO opening `<?php` tag at the top.
- writes PHP directly (assignments, `if/else`, `foreach`, `function …`).
- uses `?>` to switch to HTML output.
- writes HTML, then `<?php` to reopen PHP if more logic is needed.
- **ends with an open `<?php`** (no closing) so ScriptCase can append its code.

```php
$id = [id];
$row = array();

sc_lookup(rs, "SELECT name FROM products WHERE id = '$id'");
$name = {rs[0][0]};
?>
<html>
<body>
  <h3>Name: <?php echo $name; ?></h3>
</body>
</html>
<?php
```

Rules of thumb:
- Never start the draft with `<?php`; otherwise the pasted event has a stray `<?php`.
- Never leave a trailing `?>` at the end.
- `?>` … HTML … `<?php` may be repeated as many times as needed.
- In HTML, output ScriptCase variables with `<?php echo {var}; ?>` or `<?php echo [global]; ?>`.

---

## 2. The three variable families

| Syntax | Type | Scope / behavior | Declared in |
|--------|------|------------------|-------------|
| `[name]` | **Global variable** (session) | Persists in `$_SESSION`, passed between applications/requests. Used inside SQL in SC apps. | Application → **Global Variables** (set Type **In/Out**) |
| `{name}` | **Composite / field variable** | ScriptCase-tracked variable; scalar or multi-dim array (e.g. `{rs[0][0]}`). Available across events. In generated code it becomes `$this->name` in the main scope. | auto-detected; advanced via Programming → Attributes |
| `$name` | **Local variable** | Ordinary PHP variable, case-sensitive. Lives only in the current scope/event/method. | not declared |

Key consequences:

- Globals are the correct way to receive parameters from another application (declare as **Input**)
  or expose them (declare as **Output**). Example: `$id = [id];`. A variable used as `[x]` but not
  declared in the app may trigger an unexpected input prompt — declare every global you use.
- **Do not name a global variable the same as a table field.** For SQL identifiers use double quotes
  `"col"` instead of brackets, because `[...]` is reserved for global variables.
- You cannot pass a global directly to a function — copy to a local first:
  `$local = [my_global]; my_func($local); ... [my_global] = $returned;`
- Assignment to a global uses brackets on the left: `[name] = $value;`

---

## 3. SQL macros (the core)

> The connection parameter (when given) **must be a literal string**, never a variable nor `[global]`.
> `sc_lookup(rs, $sql, "my_connection")` is valid; `sc_lookup(rs, $sql, $conn)` is NOT.

### 3.1 `sc_lookup(dataset, $sql [, "connection"])` — simple SELECT
Best for single-row lookups and small lists. Result is an array-like dataset.

```php
$sql = "SELECT code, label FROM codes WHERE id = '" . trim($id) . "'";
sc_lookup(rs_row, $sql);

if ({rs_row} !== false && count({rs_row}) > 0) {
    $code  = {rs_row[0][0]};   // row 0, column 0
    $label = {rs_row[0][1]};   // row 0, column 1
}
```

Access patterns:
- `{dataset[row][col]}` — 0-based. `{ds[0][0]}` = first row, first selected column.
- `{dataset[index]}` / `{ds[i]}` — first column of that row.
- `count({dataset})` — number of rows (0 = empty).
- `foreach ({dataset} as $row) { echo $row[0]; }` — iterate rows.
- Guards: `{dataset} !== false` (DB error) and/or `count({dataset}) > 0`.
  `isset({ds[0][0]})` also works for "has a first row".

Loop with index (common when building a dropdown):

```php
$options = array();
sc_lookup(ds_opt, "SELECT id, name FROM categories WHERE active = 'Y' ORDER BY name");
if ({ds_opt} !== false) {
    for ($i = 0; $i < count({ds_opt}); $i++) {
        $options[ trim({ds_opt[$i][0]}) ] = trim({ds_opt[$i][1]});
    }
}
```

### 3.2 `sc_select(dataset, $sql [, "connection"])` — SELECT returning a recordset
Use when you need column names, to iterate, or to distinguish DB error vs empty.

```php
sc_select(dataset, "SELECT * FROM products WHERE trim(id) = '$id'");

if ({dataset} === false) {
    // database error (generated code exposes {dataset_erro})
} elseif ({dataset}->EOF) {
    $mode = 'add';                 // no rows
} else {
    $mode = 'edit';
    while (!{dataset}->EOF) {
        $x = {dataset}->fields['name'];
        {dataset}->MoveNext();
    }
}
{dataset}->Close();
```

Recordset API: `->fields['col']`, `->EOF`, `->MoveNext()`, `->Close()`, and `{dataset} === false`
for failure.

> **Scope rule:** in the main `onExecute` scope write the dataset reference with braces
> (`{dataset}`). Inside a **user function**, ScriptCase keeps it as a local variable, so use
> `$dataset` (`if (!$dt->EOF) { $v = $dt->fields['x']; } $dt->Close();`). Do not mix.

### 3.3 `sc_exec_sql($sql [, "connection"])` — INSERT / UPDATE / DELETE (and DDL)
No dataset is returned. Wrap writes in `try/catch` and use transactions when several statements
must succeed together.

```php
try {
    sc_exec_sql("UPDATE products SET name = '$name' WHERE id = '$id'");
} catch (Exception $e) {
    sc_error_message("Save failed: " . $e->getMessage());
}
```

### 3.4 Transactions
```php
sc_begin_trans();
try {
    sc_exec_sql($sql1);
    sc_exec_sql($sql2);
    sc_commit_trans();
} catch (Exception $e) {
    sc_rollback_trans();
    sc_error_message("Transaction rolled back.");
}
```
Only open a transaction for **write** operations. Never wrap SELECT-only code in a transaction.

### 3.5 Macro catalog (most used in blank apps)
| Macro | Purpose |
|-------|---------|
| `sc_lookup(ds, $sql [, "conn"])` | SELECT → simple array dataset |
| `sc_select(ds, $sql [, "conn"])` | SELECT → recordset (`->fields`, `->EOF`) |
| `sc_exec_sql($sql [, "conn"])` | INSERT/UPDATE/DELETE/DDL |
| `sc_begin_trans` / `sc_commit_trans` / `sc_rollback_trans` | transaction control |
| `sc_error_message($msg)` | register an application error message |
| `sc_error_exit()` | stop execution if `sc_error_message` was called |
| `sc_alert($msg [, $params])` | JavaScript alert/toast |
| `sc_confirm($msg)` | JavaScript confirm |
| `sc_redir($app_or_url [, $params])` | redirect to another application/URL |
| `sc_exit()` | force application exit |
| `sc_set_global($value)` / `sc_reset_global(...)` | register/clear session globals |
| `sc_sql_injection($value)` / `sc_sql_protect($value)` | protect a value against SQL injection / DB-aware escaping |
| `sc_date`, `sc_date_dif`, `sc_time_diff` | date/time helpers |
| `sc_mail_send`, `sc_log_add`, `sc_include_library` | email / logging / libraries |

---

## 4. Handling form POST inside a Blank app

A blank app is a single self-posting page. Generic pattern:

```php
$mode = 'add';
sc_lookup(ds, "SELECT 1 FROM products WHERE id = '$id'");
if ({ds} !== false && count({ds}) > 0) { $mode = 'edit'; }

if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_POST['action'])) {
    // text/number inputs
    $name  = isset($_POST['name']) ? trim($_POST['name']) : '';
    // checkbox / multi-select arrays -> comma string
    $tags  = !empty($_POST['tags']) ? implode(',', $_POST['tags']) : '';

    if ($mode === 'edit') {
        sc_exec_sql("UPDATE products SET name='$name', tags='$tags' WHERE id='$id'");
    } else {
        sc_exec_sql("INSERT INTO products (id, name, tags) VALUES ('$id','$name','$tags')");
    }
    // feedback + reload
    echo "<script>alert('Saved'); location.href='" . $_SERVER['PHP_SELF'] . "';</script>";
}
?>
<form method="post" action="<?php echo $_SERVER['PHP_SELF']; ?>">
    <input type="hidden" name="action" value="insert">
    ...
</form>
<?php
```

Notes:
- Detect add vs edit by querying for the key (usually the global `[id]`), not by trusting POST.
- Multi-value inputs (`name="tags[]"`) arrive as `$_POST['tags']` array → store with
  `implode(',', …)`; when loading, `explode(',', $stored)`.
- Escape values before interpolation with `sc_sql_protect(...)` / `sc_sql_injection(...)`.
- Echo user data in HTML with `htmlspecialchars($v, ENT_QUOTES)`.

---

## 5. User-defined functions and methods

```php
function get_record($id) {
    $data = array();
    sc_select(dt, "SELECT * FROM products WHERE trim(id) = '$id'");
    if (!{dt}->EOF) {           // or $dt inside a function
        $data['name'] = {dt}->fields['name'];
    }
    {dt}->Close();
    return $data;
}

$data = get_record($id);        // ScriptCase rewrites the call to a method
```

- A `function foo(...)` in the draft becomes an application method; call it normally. In generated
  code the call appears as `$this->foo(...)`.
- Inside a function `$this` is available (`$this->Db`, `$this->Ini`, …), and datasets declared with
  `sc_select`/`sc_lookup` are **local `$vars`**, not `{vars}`.
- Globals are not reliably visible inside functions — pass them as parameters (copy `[x]` to a local).
- Keep function names unique and not colliding with ScriptCase internals.

---

## 6. Output, messages and redirects

- Prefer ScriptCase macros over raw `die()`/`exit` for user-facing flow:
  `sc_error_message("...")` then `sc_error_exit();`, or `sc_alert("...")`.
- Return to the same page after save:
  `echo "<script>location.href='" . $_SERVER['PHP_SELF'] . "';</script>";`
- Jump to another application: `sc_redir("other_app");`
- Set/clear session globals: `sc_set_global($v);` / `sc_reset_global("x");`

---

## 7. HTML / CSS / JS inside the draft

- Switch with `?>`, write HTML, reopen with `<?php`.
- CDN libraries are fine (Bootstrap, jQuery, Select2, flatpickr, SweetAlert2, …) exactly as in a
  normal HTML page.
- ScriptCase bundled assets can be referenced with `$this->Ini->path_prod`, e.g.
  `<script src="<?php echo $this->Ini->path_prod; ?>/third/jquery/js/jquery.js"></script>`.
- Keep a clear separation: compute all PHP/DB values **before** `?>`, then only echo in HTML.

---

## 8. Security

- Escape output: `htmlspecialchars($v ?? '', ENT_QUOTES)`.
- Escape/parameterize SQL values: `sc_sql_protect($value)` (DB-aware) or `sc_sql_injection($value)`.
  For PostgreSQL, `pg_escape_string($value)` also works.
- Whitelist which POST keys are persisted instead of blindly looping over `$_POST`.

---

## 9. Debugging inside ScriptCase

- Quick inspect: `echo '<pre>'; print_r($var); echo '</pre>';` or `var_dump(...)`.
- Inspect POST: `echo '<pre>'; print_r($_POST); echo '</pre>';`
- Enable **Debug Mode** and **SQL Error** in the application settings to see executed SQL.
- Remember the generated page runs in an iframe/menu context; mind `target`/redirects.

---

## 10. Pitfalls and anti-patterns

- ❌ Starting the draft with `<?php` (the event already opens PHP) or leaving a final `?>`.
- ❌ Using `sc_exec_sql()` for SELECTs — use `sc_lookup()` / `sc_select()`.
- ❌ Using a variable or `[global]` as the macro connection argument — must be a **literal string**.
- ❌ Reusing the same dataset name for two concurrent result sets — give each `sc_lookup`/`sc_select`
  a unique name and `Close()` recordsets.
- ❌ Referencing `{dataset}` inside a user function (it's a local `$dataset` there).
- ❌ Assuming `$dt` in the main scope is local — ScriptCase may bind it as an application variable.
- ❌ Interpolating raw `$_POST`/`$_GET` into SQL (injection) or echoing unescaped user data (XSS).
- ❌ Naming a global variable the same as a table column.
- ❌ Wrapping SELECT-only logic in `sc_begin_trans()`.
- ❌ Editing generated `index.php` — regenerate by publishing the blank app instead.
- ❌ Building large lookup lists one `sc_lookup` per row — query once and iterate.
- ❌ Forgetting to initialize display variables to `''` before loops/conditions (undefined warnings).

---

## 11. Pre-paste checklist

1. Draft does NOT start with `<?php` and does NOT end with `?>` (ends with `<?php`).
2. Every `sc_lookup`/`sc_select` dataset has a unique name and is used consistently
   (`{ds}` in main scope, `$ds` in functions).
3. Every `sc_select` recordset is `Close()`d; add vs edit is detected from the DB, not from POST.
4. Connection names are literal strings; every global is declared in the app (In/Out).
5. Writes are wrapped in `try/catch` (+ transaction if multiple statements).
6. User input is escaped for SQL and HTML.
7. All `[globals]` used in the code exist in **Application → Global Variables**.

---

## 12. Minimal end-to-end recap

1. Read globals: `$id = [id];`
2. Load reference/dropdown data with `sc_lookup`.
3. Detect add/edit with `sc_lookup`/`sc_select` on the key.
4. Handle POST: read inputs, escape, `sc_exec_sql` inside `try/catch`.
5. Switch to HTML with `?>`, render with `<?php echo ...; ?>`.
6. End the file with an open `<?php`.

See `reference.md` for a full generic draft (globals, dropdown, add/edit, insert/update,
transaction, escaping, HTML hand-off).

