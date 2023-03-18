---
layout: post
title: "Crear un una base de datos distribuida from scratch II: API y pebbleDB"
date: 2023-02-18
tags:
  - Sistemas distribuidos
  - Storage
  - Raft
  - Golang
---

# API
Como mi idea es hacer un distributed key value store, empezare por crear una API bien simple que acepte GET, PUT y DELETE. En un futuro probably cree los metodos de GRPC (menos latencia y mayor throughput).
Al final, una base de datos solo necesita hacer dos cosas muy bien, guardar datos, y devolvermelos cuando se lo pido. 


Capitulo 1: Crear la API
- He decidido usar Gin. Parecia simple de usar, sale en un tutorial oficial de como crear un API: https://go.dev/doc/tutorial/web-service-gin y la palabra performance aparece 5 veces en la pagina de Github. 


```go
package pulse

import (
	"github.com/gin-gonic/gin"
	"net/http"
	"sync"
  "log"
)

func main() {
	r := gin.Default()
	r.GET("/:key", getKey)
	r.PUT("/:key", putKey)
	r.DELETE("/:key", deleteKey)

	r.GET("/metrics", gin.WrapH(promhttp.Handler()))

	err := r.Run("localhost:8080")
	if err != nil {
		return
	}
}

// getKey returns the value associated with the key
func getKey(c *gin.Context) {
	key := c.Param("key")
	value, err := s.Get([]byte(key))
	if err != nil {
		log.Fatal(err)
	}
	c.IndentedJSON(http.StatusOK, value)
}

// putKey adds an entry with the specified key
func putKey(c *gin.Context) {
	key := []byte(c.Param("key"))
	value := []byte(c.Param("value"))
	err := s.Set(key, value)
	if err != nil {
		log.Fatal(err)
	}
	c.IndentedJSON(http.StatusOK, key)
}

// deleteKey deletes the entry with the specified key
func deleteKey(c *gin.Context) {
	key := c.Param("key")
	s.Delete([]byte(key))
	c.IndentedJSON(http.StatusOK, key)

}
```


- La API llama a una clase que encapsula las diferentes llamadas a Pebble. 


En el caso de que me interesase trabajar solo con la parte de sistemas distribuidos probablemente usaria algo tipo RocksDB o algun otro LSM para quitarme la parte de hacer store de los datos. 

RocksDB lo he usado en el trabajo y puedo confirmar que es performant y funciona muy bien. Soy muy fan del trabajo que hizo Facebook. Por ahora podria hacer un MVP con algo ya existente. Asi que me puse a buscar algun binding the Go que me permitiera trabajar con el. 

No obstante luego me tope con Pebble, una base de datos inspirada en RocksDB escrita en Go. Asi pues puedo evitar los retos de tratar con Cgo: https://www.cockroachlabs.com/blog/the-cost-and-complexity-of-cgo/. Asi que usare Pebble.

La idea no obstante, es que en algun momento sustituire Pebble por algo mucho mas rudimentario hecho por mi (for the sake of learning). Inspirandome claramente en Pebble. 


```go
package pulse

import (
	"log"

	"github.com/cockroachdb/pebble"
)

// global store
var s *Store

// Store is a persistent key-value store based on the pebble storage engine.
type Store struct {
	store *pebble.DB
}

func createStore() (*Store, error) {
	innerDB, err := pebble.Open("db", &pebble.Options{})
	if err != nil {
		return nil, err
	}
	return &Store{
		store: innerDB,
	}, nil
}

func (d *Store) Close() error {
	return d.store.Close()
}

func (d *Store) Has(key []byte) (bool, error) {
	_, closer, err := d.store.Get(key)
	if err == pebble.ErrNotFound {
		return false, nil
	} else if err != nil {
		return false, err
	}
	err = closer.Close()
	if err != nil {
		return false, err
	}
	return true, nil
}

func (d *Store) Get(key []byte) ([]byte, error) {
	value, closer, err := d.store.Get(key)
	if err == pebble.ErrNotFound {
		return nil, err
	} else if err != nil {
		log.Fatal(err)
	}
	ret := make([]byte, len(value))
	copy(ret, value)
	err = closer.Close()
	if err != nil {
		return nil, err
	}
	return ret, nil
}

func (d *Store) Set(key []byte, value []byte) error {
	return d.store.Set(key, value, pebble.NoSync)
}

func (d *Store) Delete(key []byte) error {
	return d.store.Delete(key, nil)
}
```

y algun test:

```go
package pulse

import (
	"testing"

	"github.com/cockroachdb/pebble"
)

func TestStoreGet(t *testing.T) {
	store, _ := createStore()

	err := store.Set([]byte("adria"), []byte("cabeza"))
	if err != nil {
		t.Fatalf("Set key failed")
	}

	value, err := store.Get([]byte("adria"))

	if err != nil || string(value) != "cabeza" {
		t.Fatalf("Get stored value failed")
	}

	err = store.Delete([]byte("adria"))

	value, err = store.Get([]byte("adria"))
	if err != pebble.ErrNotFound {
		t.Fatalf("Delete key failed")
	}

}
```


