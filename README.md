# MTP_MNE-neurofeedback_codes

## class PlayerLSL(BasePlayer):
    """Class for creating a mock LSL stream.

    Parameters
    ----------
    %(player_fname)s
    chunk_size : int ``≥ 1``
        Number of samples pushed at once on the :class:`~mne_lsl.lsl.StreamOutlet`.
    %(n_repeat)s
    name : str | None
        Name of the mock LSL stream. If ``None``, the name ``MNE-LSL-Player`` is used.
    annotations : bool | None
        If ``True``, an :class:`~mne_lsl.lsl.StreamOutlet` is created for the
        :class:`~mne.Annotations` of the :class:`~mne.io.Raw` object. If ``False``,
        :class:`~mne.Annotations` are ignored and the :class:`~mne_lsl.lsl.StreamOutlet`
        is not created. If ``None`` (default), the :class:`~mne_lsl.lsl.StreamOutlet` is
        created only if the :class:`~mne.io.Raw` object has :class:`~mne.Annotations` to
        push. See notes for additional information on the :class:`~mne.Annotations`
        timestamps.

    Notes
    -----
    The file re-played is loaded in memory. Thus, large files are not recommended. Once
    the end-of-file is reached, the player loops back to the beginning which can lead to
    a small discontinuity in the data stream.

    Each time a chunk (defined by ``chunk_size``) is pushed on the
    :class:`~mne_lsl.lsl.StreamOutlet`, the last sample of the chunk is attributed the
    current time (as returned by the function :func:`~mne_lsl.lsl.local_clock`). Thus,
    the sample ``chunk[0, :]`` occurred in the  past and the sample ``chunk[-1, :]``
    occurred "now". If :class:`~mne.Annotations` are streamed, the annotations within
    the chunk are pushed on the annotation :class:`~mne_lsl.lsl.StreamOutlet`. The
    :class:`~mne.Annotations` are pushed with a timestamp corrected for the annotation
    onset in regards to the chunk beginning. However, :class:`~mne.Annotations` push is
    *not* delayed until the the annotation timestamp or until the end of the chunk.
    Thus, an :class:`~mne.Annotations` can arrived at the client
    :class:`~mne_lsl.lsl.StreamInlet` "ahead" of time, i.e. earlier than the current
    time (as returned by the function :func:`~mne_lsl.lsl.local_clock`). Thus, it is
    recommended to connect to an annotation stream with the
    :class:`~mne_lsl.lsl.StreamInlet` or :class:`~mne_lsl.stream.StreamLSL` with the
    ``clocksync`` processing flag and to always inspect the timestamps returned for
    every samples.

    .. code-block:: python

        from mne_lsl.lsl import local_clock
        from mne_lsl.player import PlayerLSL as Player
        from mne_lsl.stream import StreamLSL as Stream

        player = Player(..., annotations=True)  # file with annotations
        player.start()
        stream = Stream(bufsize=100, stype="annotations")
        stream.connect(processing_flags=["clocksync"])
        data, ts = stream.get_data()
        print(ts - local_clock())  # positive values are annotations in the "future"

    If :class:`~mne.Annotations` are streamed, the :class:`~mne_lsl.lsl.StreamOutlet`
    name is ``{name}-annotations`` where ``name`` is the name of the
    :class:`~mne_lsl.player.PlayerLSL`. The ``dtype`` is set to ``np.float64`` and each
    unique :class:`~mne.Annotations` description is encoded as a channel. The value
    streamed on a channel correspond to the duration of the :class:`~mne.Annotations`.
    Thus, a sample on this :class:`~mne_lsl.lsl.StreamOutlet` is a one-hot encoded
    vector of the :class:`~mne.Annotations` description/duration.
    """
