## Configuration

All configurations below can be made using the web configuration framework or by editing `mintable.config.json`.

`mintable.config.json` is the secret sauce – it contains all of your private tokens and is never sent to third-party servers. This file is ignored by Git – keep a backup somewhere safe as you only have 100 Plaid accounts on the free version.

> **Pro Tip:** You can use Dropbox or another trusted service to sync `mintable.config.json` across your machines. Run `ln -s <path_to_cloud_folder>/mintable.config.json .` from the repo root to symlink Mintable to the cloud version.

#### Automate Updates with a CI Provider

This repo includes config files for both [CircleCI](https://circleci.com/) and [Travis CI](https://travis-ci.com) to run builds automatically.

Most CI providers allow you to set **environment variables** to configure sensitive information (like the stuff in `mintable.config.json`). We've included a handy script to get that set up:

```
yarn export
```

Run this command and paste the result into an environment variable called `MINTABLE_CONFIG` in your CI provider of choice. Mintable will handle the rest.

> **Note:** Some CI providers (like Travis) require you to wrap this variable in single quotes, i.e. `'{ PLAID_TOKEN: "" ...}'`. If you get an error similar to `Unable to parse JSON...` when you run your CI build, give this a try.

> **Warning:** If you choose to use CircleCI, you should turn off **Pass secrets to builds from forked pull requests** under **Build Settings** > **Advanced Settings**.

#### Transaction Columns

`TRANSACTION_COLUMNS` specifies a list of [Plaid transaction properties](https://plaid.com/docs/#transactions) (using [`_.get()` syntax](https://lodash.com/docs/4.17.11#get)) to override the default automated columns. Each time you run Mintable, all the contents of these columns will be cleared and overwritten.

For example, if you only want to auto-populate the name and amount for each transaction, you could add the following line to your `mintable.config.json` file:

```
TRANSACTION_COLUMNS=["name", "amount"]
```

> **Warning:** Your mileage may vary if you choose to use additional properties outside the tested defaults (`date`, `amount`, `name`, `account`, `category.0`, `category.1`, `pending`). Proceed at your own risk, you're in uncharted territory.

#### Reference Columns

`REFERENCE_COLUMNS` specifies a list of additional, non-automated columns for your reference/bookkeeping purposes. Each time you run Mintable, the contents of these columns will be preserved.

For example, if you want to add one column to track work expenses, and another to track joint expenses shared with a partner, you could add the following line to your `mintable.config.json` file:

```
REFERENCE_COLUMNS=["work", "joint"]
```

> **Warning:** Since reference columns are not automated by Mintable, they have the potential to get out of sync with transaction data (for example, if your bank deletes a transaction, causing a row to get removed in `TRANSACTION_COLUMNS`)

#### Category Overrides

`CATEGORY_OVERRIDES` specifies a list of overrides to handle transactions that are routinely miscategorized by Plaid's servers. Overrides take the following format:

* `pattern`: [JavaScript Regular Expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp#Syntax) to test transaction names against
* `flags`: [JavaScript Regular Expression flags](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp#Syntax) (i.e. `i` for case insensitive)
* `category.0`: Override for first (top-level) category
* `category.1`: Override for second (level-2) category

For example, if you want anything matching `autopay` or `e-payment` to get categorized as `Credit Card Payment`, you could add the following line to your `mintable.config.json` file:

```
CATEGORY_OVERRIDES=[{ "pattern": ".*(autopay|e.payment).*", "flags": "i", "category.0": "Transfer", "category.1": "Credit Card Payments" }]
```

#### Transaction Provider

`ACCOUNT_PROVIDER` specifies which service to use to fetch transactions. At this time, the only possible value is `"plaid"`, but we plan to add other providers in the future.

#### Spreadsheet Provider

`SHEET_PROVIDER` specifies which service to use to automate spreadsheet updates. At this time, the only possible value is `"sheets"`, but we plan to add other providers in the future.
