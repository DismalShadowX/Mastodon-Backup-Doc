# A Comprehensive Guide to Backing Up and Restoring Your Mastodon Database

Mastodon, a popular open-source social media platform, has gained significant traction in recent years. As an administrator of a Mastodon instance, you understand the importance of safeguarding your data and ensure that your Mastodon instance stays up and running. Regularly creating backups of your Mastodon database is a fundamental part of maintaining a secure and reliable platform. In this guide, I'll walk you through the process of backing up and restoring your Mastodon database using `pg_dump` and `pg_restore` tools.

<!--more-->

* [Backing up PostgreSQL](https://docs.joinmastodon.org/admin/backups/#postgresql)
* [pg_restore from postgresql.org](https://www.postgresql.org/docs/current/app-pgrestore.html)

### Why Backup Your Mastodon Database?

It's story like this [Read about the incident at Vivaldi Social](https://thomasp.vivaldi.net/2023/07/28/what-happened-to-vivaldi-social) and other instances that experienced recovery failures or didn't test their restores or even make backups at all are a reminder and prompts Mastodon admins to reconsider their backups seriously.

Backups are essential for various reasons, including:

1 - **Disaster Recovery**: In case of hardware failure, data corruption, or accidental deletion. With a reliable backup strategy in place, you can quickly recover your Mastodon instance and its data, minimizing downtime and ensuring a seamless user experience.

2 - **Data Preservation**: Backups help safeguard user-generated content and maintain the continuity of your Mastodon community.

3 - **Upgrading and Migrating**: Before upgrading to a new Mastodon version or migrating to a different server, a backup ensures a smooth transition.

# Creating a Mastodon Database Backup

To create a backup of your Mastodon database, you can use `pg_dump`, a powerful utility for PostgreSQL databases. Follow these steps:

1- Open your terminal and login as the Mastodon user: Ensure you're logged in as the user that runs your Mastodon instance. Typically, this user is named "mastodon." You can check the database of your Mastodon from your `.env.production` file when you first created the instance or list all the databases with `\l`

![PostgreSQL databases](https://everythingbagel.me/images/postgres-list-databases-example-database.png)

2 - Run the backup command to your PostgreSQL database using `psql`:

```psql -h 127.0.0.1 -d mastodon -U mastodon -W```

Replace `127.0.0.1` with appropriate host and `mastodon` with your `DB_NAME` and `DB_USER` respectively.

You will then be prompted to enter your Mastodon database password.

Use `pg_dump` to create a backup file. This command saves your Mastodon database to a compressed tar file:

```pg_dump -U mastodon -W -F t mastodon > /var/www/backups/mastodon_backup.dump```

* -U mastodon: This specifies the PostgreSQL user to use.
* -W: This prompts for the user's password, enhancing security.
* -F t: This specifies the format of the backup as "tar."
* mastodon: This is the name of the database to back up.
* `/var/www/backups/mastodon_backup.dump`: Path where the backup file will be saved. Replace the path with the desired backup file location suitable for your setup be it local or on S3/CDN.

### Confirm your backup:
After running the command, verify that the backup file has been created successfully in the specified location. You'll need this file for the restoration process.

**Restoring from a Backup**

Should the need arise, you can restore your Mastodon database from the backup file. Follow these steps to do so:

1 - Ensure that your Mastodon instance is stopped, as you cannot restore the database while the application is running. You can do this with the following command: 

`sudo systemctl stop mastodon-web mastodon-streaming mastodon-sidekiq`

2 - Use `pg_restore` to restore your Mastodon database from the backup file:

```pg_restore -U mastodon -d mastodon < /var/www/backups/mastodon_backup.dump```

This command will load the database from the specified backup file into your Mastodon instance.

### Wait for the restoration to complete:

The restoration process may take some time, depending on the size of your database so grab some coffee ☕. Once it's finished, you should see no return from your terminal without errors, this indicates that the restore has been successfully and your Mastodon instance has been restored to the state captured in your backup.

Restart Mastodon services: `sudo systemctl restart mastodon-web mastodon-streaming mastodon-sidekiq`

### Additional Considerations

Here are some extra points to keep in mind:

1 - **Regular Backups**: Schedule regular backups to keep your data up to date.

2 - **Data Encryption**: Consider encrypting your backup files to protect sensitive user data.

3 - **Backup Storage**: Store your backup files securely, whether on-site or off-site, to ensure they are accessible when needed.

4 - **Testing Restorations**: Periodically test the restoration process on a non-production environment to verify its reliability.

5 - **Documentation**: Keep a record of your backup and restoration procedures for reference.

### Testing Restores in a Non-Production Environment

To test restores in a non-production environment, you'll need to consider a few key points:

1 - **Cloning Your Production Environment**: Set up a separate environment that mirror your production Mastodon instance. This can be done by creating a replica instance, using a copy of your data, or by setting up a testing environment with the same configuration.

2 - **Adjusting Configuration**: In your testing environment, you may need to adjust configuration settings to avoid conflicts with the existing `mastodon` database. For example, you can use a different database name or a different schema.

3 - **Restoring the Backup**: Once your testing environment is prepared, restore the backup file as described in this guide. Ensure that you adapt the database name and other settings to match your testing environment.

4 - **Testing User Access**: Confirm that user access and data integrity are preserved in the restored environment. This includes checking user accounts, posts, and other functionalities.

### Dealing with Existing `mastodon` Database

In some scenarios, users may encounter issues when attempting to restore a backup to a PostgreSQL database with an existing `mastodon` database. To address this, you may need to consider the following:

1 - **Dropping the Existing Database**: If the `mastodon` database already exists and is causing conflicts during the restoration process, you will need to drop it before performing the restore. Here's how you can drop the existing database:

`dropdb -U mastodon mastodon`

This command will remove the `mastodon` database,  After dropping it, you can proceed with the restoration.

The SQL command `DROP DATABASE mastodon;` can also be used to drop the database, but it's typically executed directly within the PostgreSQL database using `psql`. If you prefer using SQL commands, you can include this option as an alternative way to drop the database.

2 - **Using a Different Database Name**: As an alternative, you can choose to use a different database name for the restored database. In this case, you should also adjust your Mastodon configuration to point to the new database name.

To perform this test, you should create a separate environment for testing. Since the `mastodon` database already exists on your production server, you will need to create a clean environment for testing or mirrors your production database.

1 - **Create a Test Database**:
You can use the `createdb` command to do this. For example:

`createdb -U mastodon test_mastodon`

This command creates a new database called test_mastodon with the same user as your production database. Alternatively, you can do the `CREATE DATABASE mastodon` without creating user by also executed directly within the PostgreSQL database using `psql` like so:

`sudo -i -u postgres psql`

`CREATE DATABASE test_mastodon;`

2 - **Restore the Backup**: Now, you can restore your backup into the test database:

`pg_restore -U mastodon -d test_mastodon < /var/www/backups/mastodon_backup.dump`

This will load your backup into the test database without affecting your production environment.

3 - **Test the Restore**: You can perform various tests in this test environment to ensure that the restoration process is successful and the data is intact. Once you are satisfied with the test, you can drop the test database as shown in the steps above then proceed with live production.

### The Docker Method
This guide should work for docker environment, you may need to adjust PostgreSQL path in `docker-compose.yml` for PostgreSQL if needed.

Conclusion

Backing up and restoring your Mastodon database is a critical aspect of managing your Mastodon instance. By following the steps outlined in this guide, you can safeguard and confidently back up and restore your Mastodon database against data loss and ensure a smooth recovery in case of emergencies using the `pg_dump` and `pg_restore` tools, ensuring your community's data remains safe and accessible. Regularly creating and test your restores, not once, not twice, 3-2-1 rule is the best practice that ensures the long-term health, reliability, stability, secure and resilient of your Mastodon community.
