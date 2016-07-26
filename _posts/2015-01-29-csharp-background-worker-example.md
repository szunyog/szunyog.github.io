---
layout: post
title:  "C# Background Worker example"
date:   2015-01-29 20:31:49
categories: csharp
tags: [async, backgroundworker]
description: Yet an other C# BackgroundWorker sample.
---

Simple file copy example using BackgroundWorker class.


{% highlight csharp %}

public class FileAsyncCopy
{
    private string _source;
    private string _target;
    BackgroundWorker _worker;
    public FileAsyncCopy(string source, string target)
    {
        if (!File.Exists(source))
            throw new FileNotFoundException(string.Format(@"Source file was not found. FileName: {0}", source));

        _source = source;
        _target = target;
        _worker = new BackgroundWorker();
        _worker.WorkerSupportsCancellation = false;
        _worker.WorkerReportsProgress = true;
        _worker.DoWork += DoWork;
    }


    private void DoWork(object sender, DoWorkEventArgs e)
    {
        int bufferSize = 1024*512;
        using(FileStream inStream = new FileStream(_source, FileMode.Open, FileAccess.Read, FileShare.ReadWrite))
        using (FileStream fileStream = new FileStream(_target, FileMode.OpenOrCreate, FileAccess.Write))
        {
            int bytesRead = -1;
            var totalReads = 0;
            var totalBytes = inStream.Length;
            byte[] bytes = new byte[bufferSize];
            int prevPercent = 0;

            while ((bytesRead = inStream.Read(bytes, 0, bufferSize)) > 0)
            {
                fileStream.Write(bytes, 0, bytesRead);
                totalReads += bytesRead;
                int  percent = System.Convert.ToInt32(((decimal)totalReads / (decimal)totalBytes) * 100);
                if (percent != prevPercent)
                {
                    _worker.ReportProgress(percent);
                    prevPercent = percent;
                }
            }
        }
    }

    public event ProgressChangedEventHandler ProgressChanged
    {
        add { _worker.ProgressChanged += value; }
        remove { _worker.ProgressChanged -= value; }
    }

    public event RunWorkerCompletedEventHandler Completed
    {
        add { _worker.RunWorkerCompleted += value; }
        remove { _worker.RunWorkerCompleted -= value; }
    }
    public void StartAsync()
    {
        _worker.RunWorkerAsync();
    }
}


// Usage
public class Usage
{
  public void Start()
  {
    FileAsyncCopy copy = new FileAsyncCopy(@"c:\a.dat", @"c:\b.dat");
    copy.Completed += CopyCompleted;
    copy.ProgressChanged += CopyProgressChanged;
    copy.StartAsync();
  }
  private void CopyCompleted(object sender, System.ComponentModel.RunWorkerCompletedEventArgs e)
  {
    BackgroundWorker worker = sender as BackgroundWorker;
    worker.Dispose();
    // do something
  }

  private void CopyProgressChanged(object sender, System.ComponentModel.ProgressChangedEventArgs e)
  {
    // change progress bar or whatever
  }
}

{% endhighlight %}

