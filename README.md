# sweisser.github.io

## Bunch of CheatSheets

### Linux burn encrypted DVD

````
dd if=/dev/zero of=disk.img bs=1M count=4400

losetup /dev/loop1 disk.img && cryptsetup luksFormat /dev/loop1 && cryptsetup luksOpen /dev/loop1 mybackupdisk && genisoimage -R -J -joliet-long -graft-points -V backup -o /dev/mapper/mybackupdisk directory-to-backup

cryptsetup luksClose /dev/mapper/mybackupdisk && losetup -d /dev/loop1

# cryptsetup luksOpen /dev/sr0 mydvd
Enter passphrase for /dev/sr0:

# mount /dev/mapper/mydvd backmnt/
mount: block device /dev/mapper/mydvd is write-protected, mounting read-only

# umount backmnt/

# cryptsetup luksClose /dev/mapper/mydvd

````

### Rust Streams

Vector / Iterator to stream.
````rust
use futures::stream::{self, StreamExt};

let stream = stream::iter(vec![17, 19]);
assert_eq!(vec![17, 19], stream.collect::<Vec<i32>>().await);
````

