# Node-Streamlogger

## About

A node.js library for logging to multiple files

## Features

 * Supports multiple severity levels (debug, info, warn, fatal)
 * Supports writing to multiple files simultaneously
 * Supports log rotation (reopening log files on demand)
 * Emits events for most events

## Example (Taken from example.js)

    var
      sys = require('sys'),
      path = require('path'),
      http = require('http'),
      streamLogger = require('./lib/streamlogger'),
      logger = new streamLogger.StreamLogger('log1.txt','log2.txt');
       
    logger.level = streamLogger.levels.info; //Defaults to info  
    logger.emitter
      .addListener('open', function() {
        sys.puts("All streams are open");
      })
      .addListener('error', function(err,logPath) {
        sys.puts("Error: " + err + " while writing to " + logPath);
      })
      .addListener('message', function(message,levelName) {
        sys.puts("Received message: " + message + " with level " + levelName);
      })
      .addListener('loggedMessage', function(message,levelName) {
        //Only gets called for levels allowed by StreamLogger#level
        sys.puts("Logged message: " + message + " with level " + levelName);
      })
      .addListener('close', function() {
        sys.puts("All loggers closed");
      });

    //If you want to rotate logs, this will re-open the files on sighup
    process.addListener("SIGHUP", function() {
      logger.reopen();  
    });

    http.createServer(function(req, res) {
      logger.debug("DebugMsg"); //Won't get printed since log level was set to debug
      logger.info("InfoMsg");
      logger.warn("WarnMsg", function () {sys.puts("MSG flushed to kernel")});
      logger.fatal("FatalMsg");
       
      res.writeHead(200);
      res.write("Hello!");
      res.close();
    }).listen(8000);
