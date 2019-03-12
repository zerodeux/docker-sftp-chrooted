Sample configuration for chrooted SFTP in container
===================================================

Testing is easier if you copy your own SSH pubkey as Alice's one :

    cp ~/.ssh/id_rsa.pub alice.pub

Build and launch a test with :

    docker build -t onpc:sftp -f Dockerfile.sftp .
    docker run -d --rm --name sftp_test -p 2222:22/tcp onpc:sftp

Verify that SSH interactive sessions (shells) are forbidden :

    ssh -o StrictHostKeyChecking=no -p2222 alice@localhost
    ...
    This service allows sftp connections only.
    Connection to localhost closed.

Veify that SFTP accepts pubkey auth and is chrooted :

    sftp -o StrictHostKeyChecking=no -P2222 alice@localhost
    sftp> ls /etc
    Can't ls: "/etc" not found

End of test :

    docker stop sftp_test


Usage
=====

Mount the volumes you want your chrooted SFTP users to reach in their homes
(eg. as /home/alice/myvol).

If you want to expose read-only volumes, use the Docker feature for that.

If you want distinct accounts to share the read/write UID, fix Dockerfile.sftp
as :


    RUN adduser --disabled-password --uid 1000 --ingroup sftp --gecos "" alice && \
        adduser --disabled-password            --ingroup sftp --gecos "" bob && \
        sed -i -e 's/^bob:x:[0-9]\+/bob:x:1000/' /etc/passwd && \
        chown -R root: /home

Accounts 'alice' and 'bob' still have distinct homes and authorized pubkeys,
but they share the same ID. This is "illegal" (thus the 'sed' hack), but it
works fine in this context.

