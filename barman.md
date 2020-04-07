EXCEPTION: 'ascii' codec can't encode characters in position 0-4: ordinal not in range(128)
/usr/lib/python2.7/dist-packages/barman/cli.py
def main():
    """
    The main method of Barman
    """

    reload(sys)
    sys.setdefaultencoding('utf8')