+++
title = "go反向代理quickstart"
date = 2022-04-07
tags = ["go"]
categories = ["go"]
+++


main.go
```
package main

import (
	"context"
	"github.com/gorilla/mux"
	"github.com/txn2/txeh"
	"log"
	"net/http"
	"net/http/httputil"
	"net/url"
	"os"
	"os/signal"
)

var hostArr = []string{"project-01", "project-02"}

const (
	host     = "api-dev.deveye.cn"
	proxyUrl = "https://api-dev.deveye.cn/"
	address  = "127.0.0.1"
)

func main() {
	hosts, err := txeh.NewHostsDefault()
	if err != nil {
		panic(err)
	}
	hosts.AddHosts(address, hostArr)
	log.Println(hosts.Save())
	r := mux.NewRouter()
	r.Use(LoggingHandler, AuthHandler, RecoverHandler)
		r.PathPrefix("/").HandlerFunc(ReverseProxy)
	srv := &http.Server{
		Addr:    ":80",
		Handler: r,
	}
	go func() {
		if err := srv.ListenAndServe(); err != nil {
			log.Println(err)
		}
	}()
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)
	<-c
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	log.Println(srv.Shutdown(ctx))
	hosts.RemoveHosts(hostArr)
	log.Println(hosts.Save())
	log.Println("shutting down")
	os.Exit(0)
}

func ReverseProxy(w http.ResponseWriter, r *http.Request) {
	u, err := url.Parse(proxyUrl)
	if err != nil {
		log.Println(err)
	}
	proxy := httputil.NewSingleHostReverseProxy(u)
	r.Host = host
	proxy.ServeHTTP(w, r)
}
```
middleware.go
```

import (
	"bytes"
	"encoding/json"
	"errors"
	"io/ioutil"
	"log"
	"net/http"
	"time"
)

var loginUrl = proxyUrl + "login/sms"

func LoggingHandler(next http.Handler) http.Handler {
	fn := func(w http.ResponseWriter, r *http.Request) {
		t1 := time.Now()
		next.ServeHTTP(w, r)
		t2 := time.Now()
		log.Printf("[%s] %q %v", r.Method, r.URL.String(), t2.Sub(t1))
	}
	return http.HandlerFunc(fn)
}

func RecoverHandler(next http.Handler) http.Handler {
	fn := func(w http.ResponseWriter, r *http.Request) {
		defer func() {
			if err := recover(); err != nil {
				log.Printf("Recover from panic: %+v", err)
				http.Error(w, http.StatusText(500), 500)
			}
		}()
		next.ServeHTTP(w, r)
	}
	return http.HandlerFunc(fn)
}

func AuthHandler(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		token, err := getAccessToken()
		if err != nil {
			log.Fatalln(err)
		}
		r.Header.Set("X-Auth-AppId", "10003")
		r.Header.Set("X-Auth-Token", token)
		next.ServeHTTP(w, r)
	})
}

func getAccessToken() (string, error) {
	params := map[string]string{
		"phone":      "15555555551",
		"verifyCode": "888888",
		"codeId":     "111111",
	}
	marshal, _ := json.Marshal(params)
	reader := bytes.NewReader(marshal)
	request, _ := http.NewRequest("POST", loginUrl, reader)
	request.Header.Add("X-Auth-AppId", "10003")
	request.Header.Set("content-type", "application/json")
	client := &http.Client{}
	do, err := client.Do(request)
	if err != nil {
		return "", err
	}
	b, err := ioutil.ReadAll(do.Body)
	if err != nil {
		return "", err
	}
	_ = do.Body.Close()
	result := &LoginResult{}
	if err := json.Unmarshal(b, result); err != nil {
		return "", err
	}
	if result.ErrCode != "0" {
		return "", errors.New(result.Message)
	}
	return result.Data.Token, nil
}

type LoginResult struct {
	ErrCode string
	Message string
	Data    struct {
		Name  string
		Token string
		Phone string
	}
}
```
