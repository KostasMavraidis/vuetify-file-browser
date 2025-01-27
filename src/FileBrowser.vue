<template>
    <v-card class="mx-auto" :loading="loading > 0">
        <toolbar
            :path="path"
            :storages="storagesArray"
            :storage="activeStorage"
            :endpoints="endpoints"
            :axios="axiosInstance"
            :startingFolders="startingFolders"
            v-on:storage-changed="storageChanged"
            v-on:path-changed="pathChanged"
            v-on:add-files="addUploadingFiles"
            v-on:folder-created="refreshPending = true"
        ></toolbar>
        <v-row no-gutters>
            <v-col v-if="tree && $vuetify.breakpoint.smAndUp" sm="auto">
                <tree
                    :path="path"
                    :storage="activeStorage"
                    :icons="icons"
                    :endpoints="endpoints"
                    :axios="axiosInstance"
                    :refreshPending="refreshPending"
                    :showTemporary="showTemporary"
                    :startingFolders="startingFolders"
                    v-on:path-changed="pathChanged"
                    v-on:loading="loadingChanged"
                    v-on:refreshed="refreshPending = false"
                    v-on:item-changed="(e) => itemEmitted = e"
                ></tree>
            </v-col>
            <v-divider v-if="tree" vertical></v-divider>
            <v-col>
                <list
                    :path="path"
                    :storage="activeStorage"
                    :icons="icons"
                    :endpoints="endpoints"
                    :axios="axiosInstance"
                    :refreshPending="refreshPending"
                    :showTemporary="showTemporary"
                    :startingFolderPath="startingFolderPath"
                    v-on:path-changed="pathChanged"
                    v-on:loading="loadingChanged"
                    v-on:refreshed="refreshPending = false"
                    v-on:file-deleted="refreshPending = true"
                    :item="itemEmitted"
                ></list>
            </v-col>
        </v-row>
        <upload
            v-if="uploadingFiles !== false"
            :path="path"
            :storage="activeStorage"
            :files="uploadingFiles"
            :icons="icons"
            :axios="axiosInstance"
            :endpoint="endpoints.upload"
            :maxUploadFilesCount="maxUploadFilesCount"
            :maxUploadFileSize="maxUploadFileSize"
            v-on:add-files="addUploadingFiles"
            v-on:remove-file="removeUploadingFile"
            v-on:clear-files="uploadingFiles = []"
            v-on:cancel="uploadingFiles = false"
            v-on:uploaded="uploaded"
        ></upload>
    </v-card>
</template>

<script>
import axios from "axios";

import Toolbar from "./Toolbar.vue";
import Tree from "./Tree.vue";
import List from "./List.vue";
import Upload from "./Upload.vue";


const availableStorages = [
    {
        name: "",
        code: "",
        icon: "mdi-folder-multiple-outline"
    },
    {
        name: "Main",
        code: "Main",
        icon: "mdi-amazon-drive"
    }
    /*{
        name: "Dropbox",
        code: "dropbox",
        icon: "mdi-dropbox"
    }*/
];

const endpoints = {
    list: { url: "/storage/{storage}/list?path={path} ", method: "get" },
    upload: { url: "/storage/{storage}/upload?path={path}", method: "post" },
    mkdir: { url: "/storage/{storage}/mkdir?path={path}", method: "post" },
    delete: { url: "/storage/{storage}/delete{Id}", method: "delete" },
    dirdelete: { url: "/storage/{storage}/dirdelete?path={path}", method: "delete" },
    download: {url: "/storage/{storage}/download{Id}", method: "get"},
    getEmail: {url: "/ShareFiles/GetFullNames/{Name}", method: "get"},
    share: {url: "/ShareFiles/SendEmail", method: "post"},
    file: {url: "/storage/{storage}/{id}", method: "get"},
    getcategoriesAndTags: {url: "/CategoriesAndTags/GetFileTagsAndCategories/{Id}", method: "get"},
    postCategoriesAndTags: {url: "/CategoriesAndTags/UpdateFile", method: "post"},
    deleteFiles: {url: "/MoveCopyFiles/DeleteFiles", method: "post"},
    moveFiles: {url: "/MoveCopyFiles/MoveFiles", method: "put"},
    copyFiles: {url: "/MoveCopyFiles/CopyFiles", method: "post"},
    getAllFoldersAsTree: {url: "MoveCopyFiles/AllFoldersAsTree", method: "post"},
    getTemporaryFolders: {url: "TemporaryFilesAndFolders/FoldersAndFiles/{showTemporary}?path={path}", method: "get"}
};

const fileIcons = {
    zip: "mdi-folder-zip-outline",
    rar: "mdi-folder-zip-outline",
    htm: "mdi-language-html5",
    html: "mdi-language-html5",
    js: "mdi-nodejs",
    json: "mdi-json",
    md: "mdi-markdown",
    pdf: "mdi-file-pdf",
    png: "mdi-file-image",
    jpg: "mdi-file-image",
    jpeg: "mdi-file-image",
    mp4: "mdi-filmstrip",
    mkv: "mdi-filmstrip",
    avi: "mdi-filmstrip",
    wmv: "mdi-filmstrip",
    mov: "mdi-filmstrip",
    txt: "mdi-file-document-outline",
    xls: "mdi-file-excel",
    other: "mdi-file-outline"
};

export default {
    components: {
        Toolbar,
        Tree,
        List,
        Upload,
    },
    model: {
        prop: "path",
        event: "change"
    },
    props: {
        // comma-separated list of active storage codes
        storages: {
            type: String,
            default: () => availableStorages.map(item => item.code).join(",")
        },
        // code of default storage
        storage: { type: String, default: "" },
        // show tree view
        tree: { type: Boolean, default: true },
        // file icons set
        icons: { type: Object, default: () => fileIcons },
        // custom backend endpoints
        endpoints: { type: Object, default: () => endpoints },
        // custom axios instance
        axios: { type: Function },
        // custom configuration for internal axios instance
        axiosConfig: { type: Object, default: () => {} },
        // max files count to upload at once. Unlimited by default
        maxUploadFilesCount: { type: Number, default: 0 },
        // max file size to upload. Unlimited by default
        maxUploadFileSize: { type: Number, default: 0 },
        showTemporary: {type: Boolean, default: false },
        startingFolderPath: { type: String, default: "/" }
    },
    data() {
        return {
            loading: 0,
            path: "",
            activeStorage: null,
            uploadingFiles: false, // or an Array of files
            refreshPending: false,
            axiosInstance: null,
            itemEmitted: null
        };
    },
    computed: {
        storagesArray() {
            let storageCodes = this.storages.split(","),
                result = [];
            storageCodes.forEach(code => {
                result.push(availableStorages.find(item => item.code == code));
            });
            return result;
        },
        startingFolders() {
            if (this.startingFolderPath === "/") {
                return [{
                    type: "dir",
                    path: "/",
                    basename: "root",
                    extension: "",
                    name: "root",
                    children: []
                }];
            }
            const tempArray = this.startingFolderPath.split("/").filter(x=>x);
            const basename = tempArray[tempArray.length-1];
            return [{
                type: "dir",
                path: this.startingFolderPath,
                basename: basename,
                extension: "",
                name: basename,
                children: []
            }];
        }
    },
    methods: {
        loadingChanged(loading) {
            if (loading) {
                this.loading++;
            } else if (this.loading > 0) {
                this.loading--;
            }
        },
        storageChanged(storage) {
            this.activeStorage = storage;
        },
        addUploadingFiles(files) {
            files = Array.from(files);

            if (this.maxUploadFileSize) {
                files = files.filter(
                    file => file.size <= this.maxUploadFileSize
                );
            }

            if (this.uploadingFiles === false) {
                this.uploadingFiles = [];
            }
            
            if (this.maxUploadFilesCount && this.uploadingFiles.length + files.length > this.maxUploadFilesCount) {
                files = files.slice(0, this.maxUploadFilesCount - this.uploadingFiles.length);
            }

            this.uploadingFiles.push(...files);
        },
        removeUploadingFile(index) {
            this.uploadingFiles.splice(index, 1);
        },
        uploaded() {
            this.uploadingFiles = false;
            this.refreshPending = true;
        },
        pathChanged(path) {
            this.path = path;
            this.$emit("change", path);
        }
    },
    created() {
        this.activeStorage = this.storage;
        this.axiosInstance = this.axios || axios.create(this.axiosConfig);
    },
    mounted() {
        if (!this.path && !(this.tree && this.$vuetify.breakpoint.smAndUp)) {
            this.pathChanged(this.startingFolders[0].path);
        }
    }
};
</script>