To install a macOS `.pkg` file to a customized folder without admin privileges, you can follow these steps:

### Step 1: Extract the Package

You can use the built-in `pkgutil` command to extract the contents of the `.pkg` file. Open the Terminal and run the following command:

```bash
pkgutil --expand /path/to/your/package.pkg /path/to/extracted/
```

Replace `/path/to/your/package.pkg` with the path to your `.pkg` file and `/path/to/extracted/` with the directory where you want to extract the contents.

### Step 2: Locate the Payload

After extraction, navigate to the extracted directory. Inside, you will typically find a `Payload` file, which may be compressed. If it's a compressed file, you can extract it using:

```bash
cd /path/to/extracted/
mkdir payload
cd payload
gzip -d ../Payload
```

### Step 3: Manually Install the Files

Once you've extracted the files, you can manually move them to your desired location. For example, if you want to install it in a custom directory within your home folder:

```bash
mv /path/to/extracted/payload/* ~/custom_directory/
```

Make sure to create `~/custom_directory/` beforehand if it doesn't exist.

### Step 4: Clean Up

After moving the files, you can delete the extracted folder and any temporary files created during extraction:

```bash
rm -rf /path/to/extracted/
```

### Additional Notes

- **Permission Issues**: Some applications may have dependencies or require system-level access that cannot be bypassed without admin privileges. In such cases, this method may not work fully.
- **Alternative Tools**: Consider using tools like **unpkg** for more complex installations that need additional handling beyond simple file copying.
- **Manual Installation**: For applications that do not require complex installation processes, dragging and dropping them into your Applications folder (within your home directory) is often sufficient.

By following these steps, you should be able to install a `.pkg` file into a custom directory without needing admin privileges on macOS.

Citations:
[1] https://stackoverflow.com/questions/12863944/how-do-you-specify-a-default-install-location-to-home-with-pkgbuild
[2] https://www.youtube.com/watch?v=dn9Mi6t3_Lc
[3] https://apple.stackexchange.com/questions/396890/install-a-package-pkg-into-a-custom-directory
[4] https://apple.stackexchange.com/questions/454374/how-to-install-package-without-administrator-password
[5] http://s.sudre.free.fr/Software/documentation/Packages/en/Package_Settings_Customization.html
[6] https://superuser.com/questions/214072/can-i-install-programs-on-a-mac-without-admin-rights