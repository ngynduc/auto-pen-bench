FROM lucagioacchini/kali-linux-headless:latest

USER root
RUN useradd -m -s /bin/bash -G sudo vulnbot
RUN echo 'root:root' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i 's/^#\(PasswordAuthentication.*\)/\1/' /etc/ssh/sshd_config
RUN service ssh restart
RUN pip3 install pycryptodome
RUN pip3 install paramiko
RUN mkdir /root/scripts
COPY ./kali-archive-keyring.gpg /usr/share/keyrings/kali-archive-keyring.gpg
# update mirrors
RUN sed -i 's|http://http.kali.org/kali|http://kali.mirror.garr.it/mirrors/kali|g' /etc/apt/sources.list
# RUN apt-get update
RUN apt-get update 
RUN apt-get purge -y vim vim-nox vim-tiny 
RUN apt-get install -y openssh-server
# RUN apt-get install -y xxd
# Copy the adapted exploits including the flags or missing exploits
COPY adapted_exploits/openssl_heartbleed.rb /usr/share/metasploit-framework/modules/auxiliary/scanner/ssl/openssl_heartbleed.rb
COPY adapted_exploits/geoserver_unauth_rce_cve_2024_36401.rb /usr/share/metasploit-framework/modules/exploits/multi/http/geoserver_unauth_rce_cve_2024_36401.rb
COPY adapted_exploits/log4shell_scanner.rb /usr/share/metasploit-framework/modules/auxiliary/scanner/http/log4shell_scanner.rb

# Install Go
RUN apt-get install -y wget && \
    wget https://go.dev/dl/go1.20.4.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go1.20.4.linux-amd64.tar.gz && \
    rm go1.20.4.linux-amd64.tar.gz && \
    echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile && \
    echo 'export PATH=$PATH:/usr/local/go/bin' >> /root/.zshrc

# Set Go Path
ENV PATH="/usr/local/go/bin:${PATH}"

# Install Nuclei
RUN go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest && \
    cp /root/go/bin/nuclei /usr/local/bin/nuclei && \
    nuclei --update-templates

# Clean up
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
RUN gunzip /usr/share/wordlists/rockyou.txt.gz
RUN wget http://archive.ubuntu.com/ubuntu/pool/universe/s/sshpass/sshpass_1.06-1_amd64.deb && \
    dpkg -i sshpass_1.06-1_amd64.deb && \
    rm sshpass_1.06-1_amd64.deb
