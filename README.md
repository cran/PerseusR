# PerseusR

[![Travis-CI Build Status](https://travis-ci.org/jdrudolph/PerseusR.svg?branch=master)](https://travis-ci.org/jdrudolph/PerseusR)
[![CRAN_Status_Badge](http://www.r-pkg.org/badges/version/PerseusR)](https://cran.r-project.org/package=PerseusR)


Convenience functions for interop between Perseus and R.

Designed to work with the [PluginInterop](https://github.com/jdrudolph/PluginInterop) plugin
for the Perseus framework.

## Citation

If you use `PerseusR` in your projects, please cite

Rudolph, J D and Cox, J 2018, *A network module for the Perseus software for computational proteomics facilitates proteome interaction graph analysis* [doi:10.1101/447268](https://doi.org/10.1101/447268)

# Installation

Make sure to have `R >= 3.5.0` installed. Paste the following lines
into an running `R` session. You can skip the comment lines starting with `#`.

```{R}
# installing BioConductor dependencies first
install.packages('BiocManager')
BiocManager::install('Biobase')

# installing PerseusR
install.packages('PerseusR')
```

# Usage

`PerseusR` provides two functions for reading and writing files from/to Perseus.
You can use them to write simple scripts which can be used as
`MatrixProcessing` activities in Perseus. Additionally you can parse Perseus
parameters and extract their values.

an example R script that could be called though the Perseus plugin:

```{R}
# if applicable: read command-line arguments
args = commandArgs(trailingOnly=TRUE)
if (length(args) != 3) {
	stop("Should provide three arguments: parameters inFile outFile", call.=FALSE)
}
paramFile <- args[1]
inFile <- args[2]
outFile <- args[3]

library(PerseusR)
# extract parameters
parameters <- parseParameters(paramFile)
networkType <- singleChoiceParamValue(parameters, "Network type")
corFnc <- singleChoiceParamValue(parameters, "Correlation function")
power <- intParamValue(parameters, "Power")
# read data
mdata <- read.perseus(inFile)

# if additional matrices are included, the additional information like imputation can be extracted.
imputeMatrix <- imputeData(mdata)
qualityMatrix <- qualityData(mdata)

# run any kind of analysis
library(WGCNA)
net <- blockwiseModules(t(main(mdata)), power = power, corFnc = corFnc, networkType = networkType)
c1 <- net$dendrograms[[1]]
df <- as.data.frame(cbind(c1$merge, c1$height))
colnames(df) <- c('left', 'right', 'distance')

# save results to matrixData and write to file
outMdata <- matrixData(main=df)
write.perseus(outMdata, outFile)

# save results to matrixData and write to file with additional matrices

outdata <- matrixData(main = combine, imputeData = imputeMatrix, qualityData = qualityMatrix)
write.perseus(outMdata, outFile)
```

# Licensing and contributions
`PerseusR` is licensed under the MIT license. Contributions are welcome.
