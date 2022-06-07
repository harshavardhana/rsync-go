# rsync-go

![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)
[![GoDoc](https://godoc.org/github.com/minio/rsync-go?status.svg)](https://godoc.org/github.com/minio/rsync-go)

This is a pure go implementation of the rsync algorithm.

This repo in particular is a fork of Amos Wenger implementation (see the [](#Links) section for details)

### Usage

Here is a sample example (without error checking):

```go
import (
  "os"
  "bytes"

  "github.com/minio/rsync-go"
)

func main() {
  srcReader, _ := os.Open("content-v2.bin")
  defer srcReader.Close()

  rs := &rsync.RSync{}

  // here we store the whole signature in a byte slice,
  // but it could just as well be sent over a network connection for example
  sig := make([]rsync.BlockHash, 0, 10)
  writeSignature := func(bl rsync.BlockHash) error {
		sig = append(sig, bl)
		return nil
	}

	rs.CreateSignature(srcReader, writeSignature)

  targetReader, _ := os.Open("content-v1.bin")

  opsOut := make(chan rsync.Operation)
  writeOperation := func(op rsync.Operation) error {
    opsOut <- op
    return nil
  }

  go func() {
    defer close(opsOut)
    rs.CreateDelta(targetReader, writeOperation)
  }()

  srcWriter, _ := os.OpenFile("content-v2-reconstructed.bin")
  srcReader.Seek(0, os.SEEK_SET)

  rs.ApplyDelta(srcWriter, srcReader, opsOut)
}
```

### Links

- original repo: https://github.com/jbreiding/rsync-go
- paper behind the rsync algorithm: <http://www.samba.org/~tridge/phd_thesis.pdf>
