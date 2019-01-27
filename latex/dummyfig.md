```
\dummyfig{placeholder-text}{caption text}{figlabel}
\quickfig{file-to-insert}{caption text}{figlabel}
\quickfig{example-image-1}{caption text}{figlabel}

\newcommand{\dummyfig}[3]{
  \begin{figure}[!htbp]
    \centering
    \fbox{
      \begin{minipage}[c][0.2\textheight][c]{0.66\textwidth}
        \centering{#1}
      \end{minipage}
    }
    \caption{#2}
    \label{fig:#3}
  \end{figure}
}

\newcommand{\quickfig}[3]{
  \begin{figure}[!htbp]
    \centering
    \includegraphics{#1}
    \caption{#2}
    \label{fig:#3}
  \end{figure}
}
```
