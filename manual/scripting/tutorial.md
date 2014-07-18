---
---

# Scripting Tutorial

## Make *marsyas-run* stop at end of input file

By default, *marsyas-run* will run a network indefinitely. If you use it to process a soundfile, it will not automatically stop at the end of the file, but will continue to run the network even though the SoundFileSource will output a zero signal.

However, *marsyas-run* will stop when the top-level boolean control named `done` is `true` (if such a control exists). So we have to create such a control to be true when the SoundFileSource reaches the end of the file.

How do we know when the SoundFileSource has reached the end? It's control `hasData` will become `false`! Hence, let's define our top-level `done` control to be a negation of `hasData`:

    Series {
        + done = (src/hasData == false)
        -> src: SoundFileSource
    }


**Note:** An issue with the `hasData` control of `SoundFileSource` was present in Marsyas version 0.5.0-alpha, and fixed in 0.5.0-alpha1 so please make sure that you download the latest pre-built version from [SourceForge](http://sourceforge.net/projects/marsyas/) or build the latest state of the [release/0.5](https://github.com/marsyas/marsyas/tree/release/0.5) branch.


## Output to file

Marsyas would be far less useful if it couldn't cooperate with other software. One of the important facilities for cooperation is outputting computed data in a format easily usable by other software.

Perhaps the simplest form of output to file may be the Comma-Separated Values (CSV) output, implemented by the `CsvSink` MarSystem, which can easily be imported by MATLAB, Octave, LibreOffice, Microsoft Excel, etc. In the produced CSV file, rows (records) represent samples and columns (fields) represent observations. By default, fields will be separated by commas, but any separator string may be specified using the `separator` control.

    ... -> CsvSink { separator = " " }

Among other MarSystems which output to file, the following are probably most useful:

- [`WekaSink`](http://marsyas.info/assets/docs/sourceDoc/html/classMarsyas_1_1WekaSink.html)
- [`SoundFileSink`](http://marsyas.info/assets/docs/sourceDoc/html/classMarsyas_1_1SoundFileSink.html)


## Find spectral bin with largest magnitude

The [`PowerSpectrum`](http://marsyas.info/assets/docs/sourceDoc/html/classMarsyas_1_1PowerSpectrum.html) MarSystem outputs a vertical vector of spectral magnitudes - it's a matrix with one column and as many rows as there are spectral bins. In order to get a single value corresponding to the bin with the largest magnitude, one would need to look at the values in each row, and select the row with the highest value.

There is a MarSystem named [`MaxArgMax`](http://marsyas.info/assets/docs/sourceDoc/html/classMarsyas_1_1MaxArgMax.html) which selects a desired amount of *columns* with largest values. How do we get it to rather select among the *rows* of the PowerSpectrum output? The answer is the [`Transposer`](http://marsyas.info/assets/docs/sourceDoc/html/classMarsyas_1_1Transposer.html) MarSystem: it does a matrix transposition, i.e. flips the matrix around its diagonal, swapping rows for columns:

    ... -> PowerSpectrum -> Transposer -> MaxArgMax


Now, by default, `MaxArgMax` will select a single input column with largest value, but for each selected column will actually output 2 columns: one with the value and another with the index of the selected input column (like: value1, index1, ...). Let's say we want to end up with a single value which is the index only (because we care about which spectral bin has the highest value, and not what that value is). How do we drop the first output column containing the bin value?

Again, we need to select only a portion of the matrix, but this time we know the *index* that we want to select (the second column), as opposed to the *value* (the largest-valued column) in the previous case. We can use the [`Selector`](http://marsyas.info/assets/docs/sourceDoc/html/classMarsyas_1_1Selector.html) MarSystem for this purpose. Note however that Selector operates on rows (observations) rather then columns (samples), but we want to drop the first column of the `MaxArgMax` output. Well, let's just use `Transposer` again, and then drop the first row:

    ... -> MaxArgMax -> Transposer -> Selector { disable = 0 }

Voilà!

## From autocorrelation to pitch

The [`AutoCorrelation`](http://marsyas.info/assets/docs/sourceDoc/html/classMarsyas_1_1AutoCorrelation.html) MarSystem outputs the autocorrelation function of each input matrix. For pitch detection, we want to find the column index of the highest peak in that function, but not the index 0! (I will not go into details of the autocorrelation process here though...) If we used `MaxArgMax`, it would always find the value at index 0 the largest. We could remedy that by dropping a couple starting columns - but how many exactly?

You probably see why finding N highest peaks is not the same task as finding N highest values (there could be a number of highest values all around a single peak). Instead of using `MaxArgMax`, we should rather use the [`Peaker`](http://marsyas.info/assets/docs/sourceDoc/html/classMarsyas_1_1Peaker.html). It will do a better job at finding peaks by looking at groups of adjacent values, and then output a matrix where the individual peak values are preserved while all others are set to zero. We can then use `MaxArgMax` to get the highest of those:

    ... -> AutoCorrelation -> Peaker -> MaxArgMax

`Peaker` also offers more control over the selection of peaks. Read more about that in its documentation linked under its name above.
