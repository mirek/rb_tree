# Red-black tree C implementation

Based on Julienne Walker's <http://eternallyconfuzzled.com/> red-black tree implementation.

## Usage

First, grab the source into your project, you'll need just those two:

    wget https://raw.github.com/mirek/rb_tree/master/rb_tree.h
    wget https://raw.github.com/mirek/rb_tree/master/rb_tree.c

Let's say you want to store `struct iovec` objects (defined in `sys/uio.h`) with the following definition:

    struct iovec {
    	  void  *iov_base; // [XSI] Base address of I/O memory region
      	size_t iov_len;  // [XSI] Size of region iov_base points to
    };

You'll need to provide comparison callback for your object. Let's say you want to keep track
of the length of the iovec buffers, your callback should look similar to:

    int
    my_cmp_cb (struct rb_tree *self, struct rb_node *node_a, struct rb_node *node_b) {
        struct iovec *a = (struct iovec *) node_a->value;
        struct iovec *b = (struct iovec *) node_b->value;
        return (a->iov_len > b->iov_len) - (a->iov_len < b->iov_len);
    }

To create your red-black tree:

    struct rb_tree *tree = rb_tree_create(my_cmp_cb);
    if (tree) {
        
        // Use the tree here...
        for (int i = 0; i < 10; i++) {
            struct iovec *v = malloc(sizeof *v);
            v->iov_base = (void *) i;
            v->iov_len = i * i;
            
            // Default insert, which allocates internal rb_nodes for you.
            rb_tree_insert(tree, v);
        }
        
        // To f
        struct iovec *f = rb_tree_find(tree, & (struct iovec) { .iov_base = (void *) 7, .iov_len = 0 });
        if (r) {
            fprintf(stdout, "found iovec(.iov_base = %p, .iov_len = %zu)\n", f->iov_base, f->iov_len)
        } else {
            printf("not found\n");
        }

        // Dealloc call can take optional parameter to notify on each node
        // being deleted so you can free the node and/or your object:
        rb_tree_dealloc(tree, NULL);
    }

Above example will leak because malloc is not balanced with free.

If you want to iterate over elements (here in reverse order):

    struct rb_iter *iter = rb_iter_create();
    if (iter) {
        for (struct iovec *v = rb_iter_last(iter, tree); v; v = rb_iter_prev(iter)) {
            printf("- %p %zu\n", v->iov_base, v->iov_len);
        }
        rb_iter_dealloc(iter);
    }

## License

None (Public Domain).
