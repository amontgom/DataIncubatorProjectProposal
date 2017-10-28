This is a plot of the 20 most frequent bigrams in a 10,000-row subset of
a dataset of Tweets, with common stop words filtered out. It is a
continuation of the information given on "Data Incubator Proposal:
Natural Language Processing, Plot 1".  
I will be using the "tm" and "RWeka" R packages, as they contain a lot
of useful tools for this purpose.

    suppressMessages(library(tm))

    ## Warning: package 'tm' was built under R version 3.4.2

    suppressMessages(library(RWeka))

    ## Warning: package 'RWeka' was built under R version 3.4.2

    suppressMessages(library(ggplot2))

As before, due to the size of the dataset (77.6 Mb zipped, 227 Mb
unzipped) I am pre-loading a 10,000 subsample to GitHub, which this code
will be running on. The full set can be found here:
<http://help.sentiment140.com/for-students/>. I would use this code to
download the full set from the internet:

    fileUrl <- "http://cs.stanford.edu/people/alecmgo/trainingandtestdata.zip"
    temp <- tempfile()
    download.file(fileUrl, destfile = temp, mode = "wb")
    unzip(temp, "training.1600000.processed.noemoticon.csv")
    tweets <- read.csv("training.1600000.processed.noemoticon.csv", header = FALSE, nrows = 10000)
    unlink(temp)

I will be using this code instead to load the data subset:

    fileUrl <- "https://raw.githubusercontent.com/amontgom/DataIncubatorProjectProposal/master/tweets.csv"
    temp <- tempfile()
    download.file(fileUrl, destfile = temp, mode = "wb")
    tweets <- read.csv(temp, header = FALSE, nrows = 10000)
    unlink(temp)

The following is some Preprocessing for the set of Tweets, including:
extracting the tweets specifically, transforming the data into a Corpus,
and altering various data, like changing capitalization, making all
letters lower-case, stripping white space, and removing stop words.

    tweetsOnly <- tweets$V6
    corp <- Corpus(VectorSource(tweetsOnly))
    space  <- content_transformer(function(x, pattern) gsub(pattern, " ", x))
    corp <- tm_map(corp, space,"\"|/|@|\\|")
    corp <- tm_map(corp, content_transformer(tolower))
    corp <- tm_map(corp, removeNumbers)
    corp <- tm_map(corp, stripWhitespace)
    corp <- tm_map(corp, removeWords, stopwords('english'))

Here, I shift the Corpus back into a dataframe, tokenize the data, and
create my subset of bigrams, in this case the first 100 most popular
word pairs that aren't stop words.

    corpDF <- data.frame(text = unlist(corp), stringsAsFactors = FALSE)
    tokens <- NGramTokenizer(corpDF, Weka_control(min = 2, max = 2, delimiters = " \\r\\n\\t.,;:\"()?!"))
    tokens <- data.frame(table(tokens))
    biGram <- tokens[order(tokens$Freq, decreasing = TRUE),][1:100,]
    colnames(biGram) <- c("words","count")

Finally, I create a plot of the first twenty most popular word pairs in
the subset. This subset of 10,000, about 1% of the full dataset, is
fairly representative of the full one.

    ggplot(biGram[1:20,], aes(words, count)) + geom_bar(stat = "identity") +
        theme(axis.text.x = element_text(angle = 90, hjust = 1))

![](DataIncubatorProposalPlot2_files/figure-markdown_strict/unnamed-chunk-5-1.png)

The next step would be to further process out things like urls, the
creation of trigrams (no point going deeper than that, though), and
finally the application of machine-learning software to try to figure
out what people are most likely to type next.
