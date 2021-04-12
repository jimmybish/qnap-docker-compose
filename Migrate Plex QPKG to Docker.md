## Migrate Plex from QPKG to Docker

1. Stop Plex in App Center.
1. Start the container with default settings to ensure all folder mappings work. The config folder should populate with a default Library and the Web UI will allow you to login with your Plex credentials. Once everything is populated and confirmed as working, stop the container again.
    ```
    docker-compose up -d
    sleep 120
    docker-compose stop plex
    ```
1. Determine the current location of the QPKG Plex library with the following command:
    ```
    getcfg -f /etc/config/qpkg.conf PlexMediaServer Install_path
    ```
1. Copy the QPKG Library folder to the container's Library folder. This may be huge and will likely take a long time unless you compress it. I chose not to so I can have both a working QPKG installation and a docker image while I verify everything, leaving an easy rollback plan. Delete the default folder if the Plex container had been previously started and replace with the QPKG one:
    ```
    rm -rf /share/Container/plex/Library
    cp -a /share/CACHEDEV1_DATA/.qpkg/PlexMediaServer/Library/Plex\ Media\ Server /share/Container/plex/Library/Application\ Support/
    ```
1. Edit the `Preferences.xml` file in the new `Plex Media Server` folder and ensure you have a good place for automatic DB backups, mapped in `docker-compose.yml`. I chose a share on a separate volume that regularly mirrors to a cloud provider. Set `ButlerDatabaseBackupPath=/db_backups` and save the file.
1. Rebuild and start the container, and check the logs for any errors:
    ```
    docker-compose up -d
    docker-compose logs -f plex
    ```
1. Hopefully you should now have your Media Server items listed on your home screen! But they won't be accessible yet. They're still pointing to old folder locations.
    Browse to **Settings** -> **Libraries** and add the new folder locations to your libraries- internal to the container. These will be `/tv` and `/movies` and `/photos`, respectively.
1. The container configuration passes QNAP's video device for hardware transcoding with `/dev/dri:/dev/dri`. Ensure hardware acceleration is still enabled under **Settings** -> **Transcoder**.
1. Watch something from each library. Hopefully it works!
1. With everything confirmed up and running, go ahead and uninstall the old Plex QPKG instance in Application in App Center.
