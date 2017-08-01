---
title: "Using images in LaTeX"
date: 2011-04-08T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["latex", "linux", "ubuntu"]
---

Note to self and perhaps others who might be wondering why your images are not displaying in your tex document. Despite using the correct syntax:

`
    \begin{figure}[htb]
        \begin{center}
            \includegraphics{image.png}
        \end{center}
        \caption{Fig 1: image}
        \label{fig1}
    \end{figure}
`

You may find a box with with only the image path being displayed as your image. Please check that you are not using the "draft" argument at the top of your document when specifying your document class. A bit like

    \documentclass[12pt,a4paper,draft]{article}


Make sure there is no "draft" in there so that it reads

    \documentclass[12pt,a4paper]{article}


Spent about half an hour trying to figure out why everyone else's images were displaying except mine.
