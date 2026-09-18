# ScriptCase Blank Application — Reference Draft (Generic)

A complete, copy-ready skeleton for a Blank application `onExecute`. It demonstrates:
globals, a lookup, dropdown loading, add/edit detection, POST insert/update, transactions,
escaping, helper functions, and the `?>` / `<?php` HTML hand-off.

> This example is intentionally generic (`products` / `categories`). Replace names to fit your app.
> Paste the **whole file** into ScriptCase → Blank App → `onExecute`.
> It starts with PHP (no `<?php`) and ends with an open `<?php`.

```php
// ============================================================
// GLOBALS (declare in Application > Global Variables; Type In/Out)
// ============================================================
$id      = [id];
$user_id = [user_id];

$mode    = 'add';
$data    = array();
$message = '';

// ============================================================
// HELPER FUNCTIONS  (become application methods in ScriptCase)
// ============================================================
function load_options($flag) {
    $out = array();
    sc_lookup(ds_opt, "SELECT id, name FROM categories
                       WHERE active = '" . $flag . "' ORDER BY name");
    if ({ds_opt} !== false) {
        for ($i = 0; $i < count({ds_opt}); $i++) {
            $out[ trim({ds_opt[$i][0]}) ] = trim({ds_opt[$i][1]});
        }
    }
    return $out;
}

function get_product($id) {
    $row = array();
    sc_select(dt, "SELECT * FROM products WHERE trim(id) = '$id'");
    if ({dt} !== false && !{dt}->EOF) {
        $row['name']        = {dt}->fields['name'];
        $row['category_id'] = {dt}->fields['category_id'];
        $row['tags']        = {dt}->fields['tags'];
    }
    {dt}->Close();
    return $row;
}

// ============================================================
// LOAD CURRENT RECORD + DROPDOWNS
// ============================================================
sc_lookup(rs_cek, "SELECT 1 FROM products WHERE trim(id) = '$id'");
if ({rs_cek} !== false && count({rs_cek}) > 0) {
    $mode = 'edit';
    $data = get_product($id);
}

$options = load_options('Y');
$yes_no  = array('Y' => 'Yes', 'N' => 'No');

// ============================================================
// HANDLE POST (self-posting form)
// ============================================================
if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_POST['action'])) {
    // whitelist and trim inputs
    $name        = isset($_POST['name'])        ? trim($_POST['name'])        : '';
    $category_id = isset($_POST['category_id']) ? trim($_POST['category_id']) : '';
    $tags        = !empty($_POST['tags']) ? implode(',', $_POST['tags']) : '';

    // escape values for SQL (ScriptCase DB-aware protect macro)
    $name        = sc_sql_protect($name);
    $category_id = sc_sql_protect($category_id);
    // PostgreSQL alternative: pg_escape_string($value)

    try {
        sc_begin_trans();
        if ($mode === 'edit') {
            sc_exec_sql("UPDATE products
                            SET name = '$name',
                                category_id = '$category_id',
                                tags = '$tags'
                          WHERE trim(id) = '$id'");
        } else {
            sc_exec_sql("INSERT INTO products
                            (id, name, category_id, tags)
                         VALUES ('$id', '$name', '$category_id', '$tags')");
        }
        sc_commit_trans();
        $message = 'Data saved.';
    } catch (Exception $e) {
        sc_rollback_trans();
        $message = 'Save failed: ' . $e->getMessage();
    }

    // reload the same page after success
    if ($message === 'Data saved.') {
        echo "<script>alert('" . addslashes($message) . "'); location.href='" . $_SERVER['PHP_SELF'] . "?id=" . $id . "';</script>";
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Application</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
<div class="container py-3">

    <div class="card mb-3">
        <div class="card-header bg-dark text-white fw-bold">My Application</div>
        <div class="card-body">
            <div><strong>ID:</strong> <?php echo htmlspecialchars($id, ENT_QUOTES); ?></div>
            <div><strong>Mode:</strong> <?php echo ($mode === 'edit') ? 'Edit' : 'New'; ?></div>
        </div>
    </div>

    <form method="post" action="<?php echo $_SERVER['PHP_SELF']; ?>">
        <input type="hidden" name="action" value="insert">

        <div class="card mb-3">
            <div class="card-header">Details</div>
            <div class="card-body row g-3">
                <div class="col-md-6">
                    <label class="form-label">Name</label>
                    <input type="text" name="name" class="form-control"
                           value="<?php echo isset($data['name']) ? htmlspecialchars($data['name'], ENT_QUOTES) : ''; ?>">
                </div>
                <div class="col-md-6">
                    <label class="form-label">Category</label>
                    <select name="category_id" class="form-select">
                        <option value="">-- Select --</option>
                        <?php foreach ($options as $k => $v) { ?>
                            <option value="<?php echo htmlspecialchars($k, ENT_QUOTES); ?>"
                                <?php echo (isset($data['category_id']) && $data['category_id'] == $k) ? 'selected' : ''; ?>>
                                <?php echo htmlspecialchars($v, ENT_QUOTES); ?>
                            </option>
                        <?php } ?>
                    </select>
                </div>
                <div class="col-md-12">
                    <label class="form-label">Tags <small class="text-muted">(multiple allowed)</small></label>
                    <?php $selected_tags = !empty($data['tags']) ? explode(',', $data['tags']) : array(); ?>
                    <select name="tags[]" class="form-select" multiple size="4">
                        <option value="a" <?php echo in_array('a', $selected_tags) ? 'selected' : ''; ?>>Alpha</option>
                        <option value="b" <?php echo in_array('b', $selected_tags) ? 'selected' : ''; ?>>Beta</option>
                        <option value="c" <?php echo in_array('c', $selected_tags) ? 'selected' : ''; ?>>Gamma</option>
                    </select>
                </div>
            </div>
        </div>

        <?php if ($message !== '') { ?>
            <div class="alert alert-info"><?php echo htmlspecialchars($message, ENT_QUOTES); ?></div>
        <?php } ?>

        <div class="text-center">
            <button type="submit" class="btn btn-primary px-5">SAVE</button>
        </div>
    </form>
</div>
</body>
</html>
<?php
```

## Notes on this reference

- **Globals** `[id]`, `[user_id]` must be declared in Application → Global Variables.
- `load_options()` returns an assoc array for a `<select>`; call once, reuse in HTML.
- Add/edit is decided by `sc_lookup(rs_cek, ...)` — never trust the POST flag.
- Multi-select `tags[]` is stored as a comma string (`implode`); on load use `explode(',', ...)`.
- Writes use `sc_begin_trans()` … `sc_commit_trans()` / `sc_rollback_trans()` and `try/catch`.
- Output is escaped with `htmlspecialchars(..., ENT_QUOTES)`; SQL values with `sc_sql_protect()`.
- HTML is emitted after `?>`; the file ends with an open `<?php`.
