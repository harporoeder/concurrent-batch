# Concurrent STM Batch

This package facilitates batch processing based on STM. Batches are both sized, and have automatic timeout functionality.

## Example Usage

The batch handler lives inside STM. Batches should not be processed at this stage but instead pushed somewhere else for processing. In this simple example we store the results in a TMVar and process them with ```putStrLn```. Requiring the initial handler to be in STM instead of IO increases async exception safety without having to mask on a potentially blocked action.

```
import Control.Concurrent.STM
import Control.Concurrent.STM.TMVar
import Control.Concurrent.STM.Batch

main :: IO ()
main = do
  -- Create variable that batches are pushed to
  output <- newEmptyTMVarIO
  -- Create batches of 3 elements with a timeout of 10 seconds
  batcher <- newBatch 3 (Just $ fromSecs 10) (putTMVar output)
  -- Write 3 items to the batcher
  mapM_ (writeBatch batcher) [1 2 3]
  -- Get the first result batch
  batch1 <- atomically $ takeTMVar output
  -- See [3 2 1]
  putStrLn batch1
```
