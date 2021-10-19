## Snapshot

Iceberg Sanpshot 提供的功能（0.12.0）：

```java
public interface Snapshot extends Serializable {
  /**
   * Return this snapshot's sequence number.
   * <p>
   * Sequence numbers are assigned when a snapshot is committed.
   *
   * @return a long sequence number
   */
  long sequenceNumber();

  /**
   * Return this snapshot's ID.
   *
   * @return a long ID
   */
  long snapshotId();

  /**
   * Return this snapshot's parent ID or null.
   *
   * @return a long ID for this snapshot's parent, or null if it has no parent
   */
  Long parentId();

  /**
   * Return this snapshot's timestamp.
   * <p>
   * This timestamp is the same as those produced by {@link System#currentTimeMillis()}.
   *
   * @return a long timestamp in milliseconds
   */
  long timestampMillis();

  /**
   * Return all {@link ManifestFile} instances for either data or delete manifests in this snapshot.
   *
   * @return a list of ManifestFile
   */
  List<ManifestFile> allManifests();

  /**
   * Return a {@link ManifestFile} for each data manifest in this snapshot.
   *
   * @return a list of ManifestFile
   */
  List<ManifestFile> dataManifests();

  /**
   * Return a {@link ManifestFile} for each delete manifest in this snapshot.
   *
   * @return a list of ManifestFile
   */
  List<ManifestFile> deleteManifests();

  /**
   * Return the name of the {@link DataOperations data operation} that produced this snapshot.
   *
   * @return the operation that produced this snapshot, or null if the operation is unknown
   * @see DataOperations
   */
  String operation();

  /**
   * Return a string map of summary data for the operation that produced this snapshot.
   *
   * @return a string map of summary data.
   */
  Map<String, String> summary();

  /**
   * Return all files added to the table in this snapshot.
   * <p>
   * The files returned include the following columns: file_path, file_format, partition,
   * record_count, and file_size_in_bytes. Other columns will be null.
   *
   * @return all files added to the table in this snapshot.
   */
  Iterable<DataFile> addedFiles();

  /**
   * Return all files deleted from the table in this snapshot.
   * <p>
   * The files returned include the following columns: file_path, file_format, partition,
   * record_count, and file_size_in_bytes. Other columns will be null.
   *
   * @return all files deleted from the table in this snapshot.
   */
  Iterable<DataFile> deletedFiles();

  /**
   * Return the location of this snapshot's manifest list, or null if it is not separate.
   *
   * @return the location of the manifest list for this Snapshot
   */
  String manifestListLocation();

  /**
   * Return the id of the schema used when this snapshot was created, or null if this information is not available.
   *
   * @return schema id associated with this snapshot
   */
  default Integer schemaId() {
    return null;
  }
}

```

summary中的参数：

