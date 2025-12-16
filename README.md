# 2025-NetworkSecurity

## Prerequisites

- Golang

    - AMD

        ```bash
        wget https://dl.google.com/go/go1.24.5.linux-amd64.tar.gz
        sudo tar -C /usr/local -zxvf go1.24.5.linux-amd64.tar.gz
        mkdir -p ~/go/{bin,pkg,src}
        echo 'export GOPATH=$HOME/go' >> ~/.zprofile
        echo 'export GOROOT=/usr/local/go' >> ~/.zprofile
        echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.zprofile
        echo 'export GO111MODULE=auto' >> ~/.zprofile
        source ~/.zprofile
        ```

    - ARM

        ```bash
        wget https://dl.google.com/go/go1.24.5.linux-arm64.tar.gz
        sudo tar -C /usr/local -zxvf go1.24.5.linux-arm64.tar.gz
        mkdir -p ~/go/{bin,pkg,src}
        echo 'export GOPATH=$HOME/go' >> ~/.bashrc
        echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
        echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc
        echo 'export GO111MODULE=auto' >> ~/.bashrc
        source ~/.bashrc
        ```

    - MAC

        Go to go official website, and download go 1.24.5 darwin-arm64.

        ```bash
        sudo tar -C /usr/local -xzf go1.24.5.darwin-amd64.tar.gz
        export PATH=$PATH:/usr/local/go/bin
        go version
        ```

## Clone the repo

```bash
git clone https://github.com/Alonza0314/2025-NYCU-NetworkSecurity.git
```

## Homeworks

- [Homework2](homework2/README.md)
- [Homework3](homework3/README.md)
- [Homework4](homework4/README.md)
- [Homework5](homework5/README.md)
- [Midterm](midterm/README.md)

- [homework5.5](homework5.5/README.md)
- [homework6](homework6/README.md)
- [homework7](homework7/README.md)
- [homework8](homework8/README.md)